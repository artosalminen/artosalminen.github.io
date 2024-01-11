---
title: "How to sanitize NestJS API responses with Interceptor"
date: 2024-01-11
series: ["Coding", "NestJS"]
weight: 1
tags: ["Coding", "NestJS"]
author: "Arto Salminen"
---

In any API development project, ensuring that sensitive or unnecessary data is not exposed in the responses is crucial for security and efficiency. In NestJS, interceptors provide a powerful way to manipulate the flow of data. In this blog post, we'll explore how to use an interceptor to sanitize API responses by removing specific properties. We'll also look at how to write tests to ensure the interceptor behaves as expected.

Interceptors can be used to manipulate API responses in other ways as well. For instance, for creating pagination headers or similar purposes.

## Background

Before diving into the implementation, let's briefly understand the purpose of the interceptor and the scenario it addresses. The `ResponsePayloadSanitizationInterceptor` is designed to remove specific properties (e.g. `'my_super_secret_secret'`) from the API responses. These properties are often used internally by databases and might not be relevant or safe to expose to the clients consuming the API.

## Implementation

The interceptor class `ResponsePayloadSanitizationInterceptor` implements the `NestInterceptor` interface. It intercepts the response data and uses a recursive approach to remove the specified properties from objects and arrays. The interceptor is applied globally to the application using `app.useGlobalInterceptors(new ResponsePayloadSanitizationInterceptor())`.

```typescript
import { Injectable, NestInterceptor, ExecutionContext, CallHandler } from '@nestjs/common'
import { Observable } from 'rxjs'
import { map } from 'rxjs/operators'

@Injectable()
export class ResponsePayloadSanitizationInterceptor<T> implements NestInterceptor<T> {
  readonly omittedProperties = ['my_super_secret_secret']

  intercept(context: ExecutionContext, next: CallHandler): Observable<T> {
    return next.handle().pipe(map((data) => this.sanitizeProperties(data)))
  }

  private sanitizeProperties(data: any): any {
    if (Array.isArray(data)) {
      return data.map((item) => this.sanitizeProperties(item))
    }
    if (typeof data === 'object' && data !== null) {
      const sanitizedData = {}
      for (const key in data) {
        if (!this.omittedProperties.includes(key)) {
          sanitizedData[key] = this.sanitizeProperties(data[key])
        }
      }
      return sanitizedData
    }
    return data
  }
}
```

## Writing Tests

Writing tests is a crucial step to ensure that the interceptor behaves as expected. In this case, the test suite focuses on the `TestController` and verifies that the interceptor removes the specified property (`'my_super_secret_secret'`) from the API response.

```typescript
import { Controller, Get, HttpStatus, INestApplication } from '@nestjs/common'
import { Test } from '@nestjs/testing'
import { Public } from 'nest-keycloak-connect'
import * as request from 'supertest'

import { ResponsePayloadSanitizationInterceptor } from './response-payload-sanitization.interceptor'
import { AppModule } from '../../app.module'

@Public()
@Controller()
class TestController {
  // eslint-disable-next-line class-methods-use-this
  @Get('/api/test')
  test() {
    return {
      results: [
        {
          my_super_secret_secret: '1',
          value: 'hey',
          foo: {
            my_super_secret_secret: '2',
            value: 'hoy',
          },
        },
      ],
    }
  }
}

describe('ResponsePayloadSanitizationInterceptor', () => {
  let app: INestApplication

  beforeEach(async () => {
    app = (
      await Test.createTestingModule({
        imports: [AppModule],
        controllers: [TestController],
      }).compile()
    ).createNestApplication()
    app.useGlobalInterceptors(new ResponsePayloadSanitizationInterceptor())
  })

  it('should remove all secret properties from response', async () => {
    await app.init()
    const response = await request(app.getHttpServer()).get('/api/test')
    expect(response.status).toBe(HttpStatus.OK)
    expect(JSON.stringify(response.body)).toBe(
      JSON.stringify({
        results: [
          {
            value: 'hey',
            foo: {
              value: 'hoy',
            },
          },
        ],
      })
    )
  })

  afterEach(async () => {
    await app.close()
  })
})

```

The test case `should remove all secret properties from response` initializes the NestJS application, sends a request to the `/api/test` endpoint, and checks if the interceptor has successfully removed the specified properties from the response. The `supertest` library is used to make HTTP requests and assert the expected behavior.

## Conclusion

Using interceptors in NestJS provides a clean and reusable way to modify the behavior of your API. The `ResponsePayloadSanitizationInterceptor` showcased here is a practical example of how interceptors can be used to sanitize API responses by removing specific properties. Writing tests ensures that the interceptor functions correctly and provides a safety net for future changes.

By incorporating this interceptor into your NestJS application, you can enhance the security and cleanliness of your API responses, making them more suitable for consumption by client applications.