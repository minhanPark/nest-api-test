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

## dto라는 것을 만들고 유효성 검사를 할 수 있다.

dto는 data transfer object이다. 데이터를 전달할 때 형식을 말하는 것 같다.

```ts
import { IsString, IsNumber } from 'class-validator';

export class CreateMovieDto {
  @IsString()
  readonly title: string;

  @IsNumber()
  readonly year: number;

  @IsString({ each: true })
  readonly genres: string[];
}
```

위와 같은 형식으로 할 수 있음. dto만 있으면 되는 것이 아니라 pipe 설정을 해주어야함

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
    }),
  );
  await app.listen(3000);
}
bootstrap();
```

만약 whitelist만 true면 dto에 설정되지 않은 값이 들어갈 땐 해당 값이 제거되고 저장되고, forbidNonWhitelisted이 true면 설정되지 않은 값이 들어갔을 땐 400을 던진다.(forbidNonWhitelisted은 whitelist가 true가 되었을 때 사용가능)  
**transform(자동 형변환)**  
네트워크를 통해 들어오는 payload는 일반 javascript 객체인데, validationPipe는 payload를 dto 클래스에 따라 유형이 지정된 객체로 자동 변환할 수 있습니다. 자동 변환을 활성화하려면 transform을 true로 설정하면 된다.

## @nestjs/mapped-types

해당 모듈을 통해서 기존의 클래스를 변경시킬 수 있다.

```js
export class UpdateMovieDto extends PartialType(CreateMovieDto) {}
```

UpdateMovieDto는 CreateMovieDto와 모든 속성이 같은데 다 optional이라는 특징이 있다. 그럴경우엔 위처럼 쓰면 된다.

## Module과 DI

하나의 앱은 여러 모듈로 구성되어 있다.

```ts
@Module({
  imports: [MoviesModule],
  controllers: [AppController],
  providers: [],
})
export class AppModule {}
```

MoviesModule은 MoviesController와 MoviesService로 구성되어 있다. 모듈은 imports로 추가할 수 있다.  
Di는 우리가 모듈에 추가해주면 알아서 nest가 추가해주는 형태를 가리킨다.

```ts
@Controller('movies')
export class MoviesController {
  constructor(private readonly moviesService: MoviesService) {}
}
```

즉 위처럼 movieService를 갖고올 수 있었던 이유는 module에서 설정해줬기 때문이다.
