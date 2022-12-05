# nest api test

## nest는 구조 및 아키텍쳐를 가진 백엔드 프레임워크

기본적으로 nestjs를 시작하면 main.ts가 있다.

```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

AppModule 하나를 가져와서, 3천번에 listen 시킨다. 즉 root 모듈 하나만 가지고 온다.

```ts
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

AppModule에는 controller와 providers가 모이게 된다.  
**controller는 express에서 라우터의 역할**을 한다. 어떤 url에 실행할 함수를 매칭 시킨다.

```ts
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

여기서 중요한건 실제로 로직을 실행하는 부분은 service에 있다는 것이다. 그래서 this.appService.getHello 등으로 불러와서 실행시킨다.

## controller에서 쓸 수 있는 데코레이션

```
GET ?year=2023 => @Query('year')
GET /id        => @Param(":id")
POST /movies   => @Body()
//json
{
  "title": "movie title",
  "year" : "movie year"
}
```

## 기본적으로 예외처리할 수 있도록 nest에서 제공해주는 것들이 있다.

```js
import { NotFoundException } from '@nestjs/common';

throw new NotFoundException('에러 텍스트');
```

위와 같은 형태로 던지면 됨.
