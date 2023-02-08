---
title: "How to use ConfigService in NestJS in application bootstrap"
date: 2023-02-09
series: ["NestJS", "coding"]
weight: 1
tags: ["NestJS", "coding", "TypeScript", "Microservices"]
author: "Arto Salminen"
---

You might want to access the configuration, for example to set the microservice
configuration based on the values from the configuration service.

One way to do this is to create an application context from the `AppModule`.

```typescript
  const appContext = await NestFactory.createApplicationContext(BootstrapConfigModule)
  const configService = appContext.get(ConfigService)
  const SERVICE_PORT = configService.get('SERVICE_PORT')
  appContext.close()
```

But a better idea is to use a "temporary" module to avoid double instantiation of the whole app.
See a full example below.

```typescript
import { Module } from '@nestjs/common'
import { ConfigService } from '@nestjs/config'
import { NestFactory } from '@nestjs/core'
import { MicroserviceOptions, Transport } from '@nestjs/microservices'

import { AppModule } from './app.module'

// Bootstrap configuration module is needed to avoid
// doing the DI resolution twice
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
class BootstrapConfigModule {}

async function bootstrap() {
  const appContext = await NestFactory.createApplicationContext(BootstrapConfigModule)
  const configService = appContext.get(ConfigService)
  const SERVICE_PORT = configService.get('SERVICE_PORT')
  appContext.close()
  const app = await NestFactory.createMicroservice<MicroserviceOptions>(AppModule, {
    transport: Transport.TCP,
    options: {
      port: SERVICE_PORT,
    },
  })
  await app.listen()
}
bootstrap()
```

Why would you need to use the configuration service instead of `process.env` you might ask.
Well, while that may work, it doesn't offer any type safety or support for fallbacks.