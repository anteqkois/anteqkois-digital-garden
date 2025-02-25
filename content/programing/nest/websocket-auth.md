---
title: Auth for NestJS WebSocket connection
tags:
  - dev
  - nestjs
  - websocket
  - auth
  - security
  - passportjs
---

In NestJS WebSocket you can protect logic in few ways:

## Add 'CORS' to websocket
```ts
@WebSocketGateway({
  cors: {
    origin: [process.env['FRONTEND_HOST']],
    credentials: true,
  },
  withCredentials: true,
  cookie: true,
  connectTimeout: 10_000,
  namespace: WEBSOCKET_NAMESPACE.XYZ,
})
```

## Durning `handleConnection` call inside websocket gateway using JWT tokens (Here is used Socekt.io for handle websocket logic)
```ts
  handleConnection(client: Socket, ...args: any[]): void | Socket {
    if (typeof client.handshake.headers.cookie !== 'string') return client.disconnect(true)

    const jwtPayload = this.authService.verifyJwt(parseCookieString(client.handshake.headers.cookie)[Cookies.ACCESS_TOKEN])
    if (!jwtPayload) return client.disconnect(true)

    this.logger.log(`#handleConnection ${client.id}`)
  }
```

## Using simple NestJS Guard
```ts
  @UseGuards(JwtCookiesWebsocketAuthGuard)
  @UseFilters(new AllExceptionsWebsocketFilter())
  @SubscribeMessage(WEBSOCKET_EVENT.XYZ)
  async handleXYZEvent(@ConnectedSocket() client: Socket, @MessageBody() data: XYZWSInput) {
    // logic...
    client.emit(WEBSOCKET_EVENT.XYZ, response)
  }
```

`JwtCookiesWebsocketAuthGuard` logic with `passportJS`
```ts
import { ExecutionContext, Injectable, UnauthorizedException } from '@nestjs/common'
import { Reflector } from '@nestjs/core'
import { AuthGuard } from '@nestjs/passport'
import { Observable } from 'rxjs'
import { Socket } from 'socket.io'
import { IS_PUBLIC_KEY } from '../decorators/public.decorator'

@Injectable()
export class JwtCookiesWebsocketAuthGuard extends AuthGuard('jwt-websocket') {
  constructor(private reflector: Reflector) {
    super()
  }

  override canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [context.getHandler(), context.getClass()])
    if (isPublic) {
      return true
    }
    return super.canActivate(context)
  }

  override getRequest<T = any>(context: ExecutionContext): T {
    return context.switchToWs().getClient<Socket>().handshake as any
  }

  override handleRequest(err: any, user: any, info: any) {
    if (err || !user) {
      throw err || new UnauthorizedException('Invalid authorization token')
    }
    return user
  }
}
```

`JwtCookiesWebsocketAuthGuard` logic plain NestJS
```ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { Observable } from 'rxjs';
import { Socket } from 'socket.io';
import { AuthService } from '../auth.service';
import { MaxDataJwtToken } from '../types';

@Injectable()
export class JwtCookiesWebsocketAuthGuard implements CanActivate {
	constructor(private reflector: Reflector) {}

	static retriveAuthPayload(client: Socket): MaxDataJwtToken | null {
		if (typeof client.handshake.headers.authorization !== 'string') return null;

		// token can be stored in headers, cookies or predevined `auth` object in socket.io handshake object
		const token = AuthService.retriveTokenFromBearer(client.handshake.headers.authorization);
		if (!token) return null;

		const jwtPayload = AuthService.decodeJwt(token);
		return jwtPayload.sub ? jwtPayload : null;
	}

	canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
		const client = context.switchToWs().getClient<Socket>();
		const userPayload = JwtCookiesWebsocketAuthGuard.retriveAuthPayload(client);
		return !!userPayload;
	}
}
```

## All together

```ts
import { Cookies, FlowRunWSInput, WEBSOCKET_EVENT, WEBSOCKET_NAMESPACE, parseCookieString } from '@xyz/shared'
import { Logger, UseFilters, UseGuards } from '@nestjs/common'
import { ConnectedSocket, MessageBody, OnGatewayConnection, OnGatewayDisconnect, SubscribeMessage, WebSocketGateway } from '@nestjs/websockets'
import { Socket } from 'socket.io'
import { JwtCookiesWebsocketAuthGuard } from '../../../lib/auth/guards/jwt-cookies-websocket-auth.guard'
import { AllExceptionsWebsocketFilter } from '../../../lib/nest-utils'
import { AuthService } from '../../users/auth/auth.service'

@WebSocketGateway({
  cors: {
    origin: [process.env['FRONTEND_HOST']],
    credentials: true,
  },
  withCredentials: true,
  cookie: true,
  connectTimeout: 10_000,
  namespace: WEBSOCKET_NAMESPACE.XYZ,
})
export class WebSocketService implements OnGatewayConnection, OnGatewayDisconnect {
  private readonly logger = new Logger(WebSocketService.name)

  constructor(
    private authService: AuthService,
  ) {}

  handleConnection(client: Socket, ...args: any[]): void | Socket {
    if (typeof client.handshake.headers.cookie !== 'string') return client.disconnect(true)

    const jwtPayload = this.authService.verifyJwt(parseCookieString(client.handshake.headers.cookie)[Cookies.ACCESS_TOKEN])
    if (!jwtPayload) return client.disconnect(true)

    this.logger.log(`#handleConnection ${client.id}`)
  }

  handleDisconnect(client: Socket) {
    this.logger.log(`#handleDisconnect ${client.id}`)
  }

  @UseGuards(JwtCookiesWebsocketAuthGuard)
  @UseFilters(new AllExceptionsWebsocketFilter())
  @SubscribeMessage(WEBSOCKET_EVENT.XYZ)
  async handleXYZEvent(@ConnectedSocket() client: Socket, @MessageBody() data: XYZWSInput) {
    // logic...
    client.emit(WEBSOCKET_EVENT.XYZ, response)
  }
}
```

### LinkedIn post content

🚀 𝘽𝙤𝙤𝙨𝙩 𝙔𝙤𝙪𝙧 𝙉𝙚𝙨𝙩𝙅𝙎 𝙒𝙚𝙗𝙎𝙤𝙘𝙠𝙚𝙩 𝙎𝙚𝙘𝙪𝙧𝙞𝙩𝙮 🔒 [Orginal Notes](https://anteqkois.linkerry.com/programing/nest/websocket-auth)

𝙎𝙚𝙘𝙪𝙧𝙞𝙣𝙜 𝙮𝙤𝙪𝙧 𝙒𝙚𝙗𝙎𝙤𝙘𝙠𝙚𝙩 𝙘𝙤𝙣𝙣𝙚𝙘𝙩𝙞𝙤𝙣𝙨 𝙞𝙣 𝙉𝙚𝙨𝙩𝙅𝙎 𝙞𝙨 𝙚𝙨𝙨𝙚𝙣𝙩𝙞𝙖𝙡 𝙛𝙤𝙧 𝙥𝙧𝙤𝙩𝙚𝙘𝙩𝙞𝙣𝙜 𝙮𝙤𝙪𝙧 𝙙𝙖𝙩𝙖 𝙖𝙣𝙙 𝙚𝙣𝙨𝙪𝙧𝙞𝙣𝙜 𝙩𝙝𝙚 𝙧𝙞𝙜𝙝𝙩 𝙪𝙨𝙚𝙧𝙨 𝙘𝙖𝙣 𝙘𝙤𝙣𝙣𝙚𝙘𝙩. 𝙃𝙚𝙧𝙚’𝙨 𝙝𝙤𝙬:

1️⃣ **𝘾𝙤𝙧𝙨 𝙁𝙤𝙧 𝙒𝙚𝙗𝙎𝙤𝙘𝙠𝙚𝙩:**
```ts
@WebSocketGateway({
  cors: {
    origin: [process.env['FRONTEND_HOST']],
    credentials: true,
  },
  withCredentials: true,
  cookie: true,
  connectTimeout: 10_000,
  namespace: WEBSOCKET_NAMESPACE.XYZ,
})
```

2️⃣ **𝙅𝙒𝙏 𝘼𝙪𝙩𝙝 𝙞𝙣 𝙃𝙖𝙣𝙙𝙡𝙚𝘾𝙤𝙣𝙣𝙚𝙘𝙩𝙞𝙤𝙣:**
```ts
handleConnection(client: Socket, ...args: any[]): void | Socket {
  if (typeof client.handshake.headers.cookie !== 'string') return client.disconnect(true)

  const jwtPayload = this.authService.verifyJwt(parseCookieString(client.handshake.headers.cookie)[Cookies.ACCESS_TOKEN])
  if (!jwtPayload) return client.disconnect(true)

  this.logger.log(`#handleConnection ${client.id}`)
}
```

3️⃣ **𝙐𝙨𝙚 𝙉𝙚𝙨𝙩𝙅𝙎 𝙂𝙪𝙖𝙧𝙙:**
```ts
@UseGuards(JwtCookiesWebsocketAuthGuard)
@UseFilters(new AllExceptionsWebsocketFilter())
@SubscribeMessage(WEBSOCKET_EVENT.XYZ)
async handleXYZEvent(@ConnectedSocket() client: Socket, @MessageBody() data: XYZWSInput) {
  // logic...
  client.emit(WEBSOCKET_EVENT.XYZ, response)
}
```

4️⃣ **𝙅𝙬𝙩𝘾𝙤𝙤𝙠𝙞𝙚𝙨𝙒𝙚𝙗𝙨𝙤𝙘𝙠𝙚𝙩𝘼𝙪𝙩𝙝𝙂𝙪𝙖𝙧𝙙 𝙬𝙞𝙩𝙝 𝙋𝙖𝙨𝙨𝙥𝙤𝙧𝙩𝙅𝙎:**
```ts
@Injectable()
export class JwtCookiesWebsocketAuthGuard extends AuthGuard('jwt-websocket') {
  constructor(private reflector: Reflector) {
    super()
  }

  override canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [context.getHandler(), context.getClass()])
    if (isPublic) {
      return true
    }
    return super.canActivate(context)
  }

  override getRequest<T = any>(context: ExecutionContext): T {
    return context.switchToWs().getClient<Socket>().handshake as any
  }

  override handleRequest(err: any, user: any, info: any) {
    if (err || !user) {
      throw err || new UnauthorizedException('Invalid authorization token')
    }
    return user
  }
}
```

💡 **𝘼𝙡𝙡 𝙩𝙤𝙜𝙚𝙩𝙝𝙚𝙧:**
```ts
@WebSocketGateway({
  cors: {
    origin: [process.env['FRONTEND_HOST']],
    credentials: true,
  },
  withCredentials: true,
  cookie: true,
  connectTimeout: 10_000,
  namespace: WEBSOCKET_NAMESPACE.XYZ,
})
export class WebSocketService implements OnGatewayConnection, OnGatewayDisconnect {
  private readonly logger = new Logger(WebSocketService.name)

  constructor(
    private authService: AuthService,
  ) {}

  handleConnection(client: Socket, ...args: any[]): void | Socket {
    if (typeof client.handshake.headers.cookie !== 'string') return client.disconnect(true)

    const jwtPayload = this.authService.verifyJwt(parseCookieString(client.handshake.headers.cookie)[Cookies.ACCESS_TOKEN])
    if (!jwtPayload) return client.disconnect(true)

    this.logger.log(`#handleConnection ${client.id}`)
  }

  handleDisconnect(client: Socket) {
    this.logger.log(`#handleDisconnect ${client.id}`)
  }

  @UseGuards(JwtCookiesWebsocketAuthGuard)
  @UseFilters(new AllExceptionsWebsocketFilter())
  @SubscribeMessage(WEBSOCKET_EVENT.XYZ)
  async handleXYZEvent(@ConnectedSocket() client: Socket, @MessageBody() data: XYZWSInput) {
    // logic...
    client.emit(WEBSOCKET_EVENT.XYZ, response)
  }
}
```

🛡️ 𝙎𝙩𝙖𝙮 𝙨𝙚𝙘𝙪𝙧𝙚 𝙖𝙣𝙙 𝙚𝙣𝙨𝙪𝙧𝙚 𝙤𝙣𝙡𝙮 𝙖𝙪𝙩𝙝𝙤𝙧𝙞𝙯𝙚𝙙 𝙪𝙨𝙚𝙧𝙨 𝙘𝙖𝙣 𝙖𝙘𝙘𝙚𝙨𝙨 𝙮𝙤𝙪𝙧 𝙉𝙚𝙨𝙩𝙅𝙎 𝙬𝙚𝙗𝙨𝙤𝙘𝙠𝙚𝙩𝙨! 🛡️

#𝙉𝙚𝙨𝙩𝙅𝙎 #𝙒𝙚𝙗𝙎𝙤𝙘𝙠𝙚𝙩 #𝙎𝙚𝙘𝙪𝙧𝙞𝙩𝙮 #𝘼𝙪𝙩𝙝 #𝙅𝙒𝙏 #𝘾𝙤𝙙𝙞𝙣𝙜