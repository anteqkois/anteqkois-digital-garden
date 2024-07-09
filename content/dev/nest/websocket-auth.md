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

### Add 'CORS' to websocket
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

### Durning `handleConnection` call inside websocket gateway using JWT tokens (Here is used Socekt.io for handle websocket logic)
```ts
  handleConnection(client: Socket, ...args: any[]): void | Socket {
    if (typeof client.handshake.headers.cookie !== 'string') return client.disconnect(true)

    const jwtPayload = this.authService.verifyJwt(parseCookieString(client.handshake.headers.cookie)[Cookies.ACCESS_TOKEN])
    if (!jwtPayload) return client.disconnect(true)

    this.logger.log(`#handleConnection ${client.id}`)
  }
```

### Using simple NestJS Guard
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

`JwtCookiesWebsocketAuthGuard` logic plain
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