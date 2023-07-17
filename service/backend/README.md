# 널스 에듀 강의수강 서버

## 0. 목차

1. [프로젝트 개요](#1.-프로젝트-개요)
2. [프로젝트의 범위](#2.-프로젝트의-범위)
3. [기술 스택](#3.-기술-스택)
4. [프로젝트 설치/실행](#4.-프로젝트-설치/실행)
5. [인프라 구조](#5.-인프라-구조)
6. [데이터베이스 구조](#6.-데이터베이스-구조)
7. [핵심 기능 설명](#7.-핵심-기능-설명)
8. [API 문서](#8.-API-문서)
9. [기타 프로젝트 관련 설명](#9.-기타-프로젝트-관련-설명)

<a name="1.-프로젝트-개요"></a>
## 1. 프로젝트 개요
본 서버는 널스에듀의 온라인강의 수강을 위해 개발되었습니다.
백오피스로 업로드한 동영상을 수강신청, 결제를 통해 수강할 수 있는 환경을 제공합니다.

<a name="2.-프로젝트의-범위"></a>
## 2. 프로젝트의 범위

1. 회원가입, 로그인
    - AccessToken을 이용한 로그인 기능
    - 동시접속 제한 기능
2. 강의 탐색
    - 카테고리별 강의 목록 조회
    - 강의 수강 신청
    - 수강 신청 내역 조회
    - 결제 (수강 상태만 변경되도록 동작)
3. 강의 시청
    - 강의 상태 변경 (수강신청(PENDING), 수강중(INPROGRESS), 수강완료(COMPLETED))
    - 강의 시청 기록 저장 (시청 시간, 시청 완료 여부)
    - 강의 시청 제한 (시청 시간의 2배 시간 시청하면 수강 불가)

<a name="3.-기술-스택"></a>
## 3. 기술 스택

- 언어

| 언어         | 버전      |
|------------|---------|
| Typescript | 4.9.5   |
| NodeJS     | 18.16.0 |

- 프레임워크

| 프레임워크   | 버전    |
|---------|-------|
| NestJS  | 9.0.0 |

- 데이터베이스 관련

| 데이터베이스              | 버전     |
|---------------------|--------|
| Amazon RDS Postgres | 15.2   |
| TypeORM             | 0.3.16 |

- 라이브러리

| 라이브러리                     | 버전    |                                                                                                   |
|---------------------------|-------|---------------------------------------------------------------------------------------------------|
| passport                  | 0.6.0 | 로그인 관련 라이브러리                                                                                      |
| passport-jwt              | 4.0.1 | 로그인 관련 라이브러리                                                                                      |
| @js-joda/core             | 5.5.3 | [시간 관련 라이브러리](https://jojoldu.tistory.com/600)                                                    |
| @types/js-yaml            | 4.0.5 | 환경변수 yaml로 관리할 수 있도록 도와주는 라이브러리                                                                   |
| js-yaml                   | 4.1.0 | 환경변수 yaml로 관리할 수 있도록 도와주는 라이브러리                                                                   |
| cpx                       | 1.5.0 | 환경변수 yaml 빌드 도와주는 도구                                                                              |
| typeorm-naming-strategies | 4.1.0 | TypeORM 사용시 Entity가 DB 저장될때 Snake 형식으로 자동 변환해주는 라이브러리                                             |
| typeorm-transactional     | 0.4.1 | TypeORM 0.3 사용시 Service 계층에서 트랜잭션을 걸어주는 역할 [링크](https://github.com/Aliheym/typeorm-transactional) |

<a name="4.-프로젝트-설치/실행"></a>
## 4. 프로젝트 설치/실행

1. 프로젝트 클론

```bash
git clone {주소}
```

2. 패키지 설치

```bash
yarn install
```

3. 로컬에서 프로젝트 실행 (터미널, cmd)

```bash
yarn start
```

4. 환경별 환경변수 설정
- 환경변수는 .env파일이 아니라 yaml 파일로 관리하고 있습니다.
- 하기의 순서로 구현했습니다.

    <details>
    <summary> 1. (prerequisite) 패키지 설치</summary>

    ```shell
    $ yarn add @nestjs/config
    $ yarn add js-yaml
    $ yarn add @types/js-yaml --dev
    $ yarn add cpx # 파일 copy 도구
    ```
    </details>

    <details>
    <summary> 2. 설정 파일 작성 </summary>

    - config 설정 ./config/configuration.ts에 위치
        - 환경변수는 process.env.ENV로 관리
        - 환경변수는 local, test, prod로 구분

    ```typescript
    import { readFileSync } from 'fs';
    import * as yaml from 'js-yaml';
    import { join } from 'path';
    
    const env = process.env.ENV || 'local'; // 로컬, 운영, 개발 등 환경변수 분기 부분 
    const YAML_CONFIG_FILENAME = `config.${env}.yaml`;
    
    export default () => {
      return yaml.load(
        readFileSync(join(__dirname, YAML_CONFIG_FILENAME), 'utf8'),
      ) as Record<string, any>;
    };
    ```
    </details>

    <details>
    <summary> 3. yaml 파일에 환경변수 작성 </summary>

    - yaml 파일은 ./config/config.{env}.yaml에 위치
    - 아래의 예시는 config.local.yaml 파일
    ```yaml
    env: local
    
    service:
      name: 'nurseedu-backend'
    
    http:
      host: 'localhost'
    port: 3000
    
    jwt: # 토큰 관련 설정
    accessToken:
      secret: 'devcamp'
    expiresIn: '14d'
    
    db: # DB 관련 설정
    postgres:
      type: 'postgres'
    host: 'host'
    port: 5432
    databaseName: 'postgres'
    username: 'postgres'
    password: 'nurseedu'
    schema: 'nurse_edu'
    synchronize: false # DB 스키마 자동 동기화 옵션 (프로덕션에서는 절때로 false)
    logging: true # SQL 로깅 옵션
    connectTimeout: 30000 # 30s connection timeout
    queryTimeout: 10000 # 10s query timeout
    ```
    </details>

    <details>
    <summary> 4. build script 작성 </summary>

    - package.json
    ```json
    // dist 디렉토리에 yaml 파일 복사 명령
    "copy-files": "cpx \"config/*.yaml\" dist/config/",
    "prebuild": "rimraf dist",
    "build": "yarn run copy-files && nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "yarn run copy-files && nest start",
    "start:dev": "yarn run copy-files && nest start --watch",
    
    ```
    - nest-cli.json
        - nest bulld 시 dist 디렉토리 삭제, 생성이 자동입니다.
        - cpx를 사용해서 yaml 파일을 복사했는데 dist 디렉토리를 삭제하고 생성하며 파일이 사라지는 문제가 있었습니다.
        - 아래 옵션을 false로 입력하고  `"prebuild": "rimraf dist",` 스크립트로 dist를 수동 삭제하도록 설정했습니다.
    ```json
    "compilerOptions": {
      "deleteOutDir": false
    }
    ```
    </details>

    <details>
    <summary> 5. 환경변수 사용 </summary>

    - 프로젝트 내에서 yaml에 작성한 변수가 필요할때 아래와 같이 사용하면 됩니다.
    ```typescript
    const {
      http: { port },
    } = configuration();
    ```
    </details>

Reference

- https://docs.nestjs.com/techniques/configuration
- https://codegear.tistory.com/82?category=991389

<a name="5.-인프라-구조"></a>
## 5. 인프라 구조

- 인프라

<img width="800" alt="image" src="https://github.com/kingseungil/teamsparta-devcamp-nurseedu/assets/109774037/d5c2aeb6-9404-43bd-afa4-afd2a0cfb528">



<a name="6.-데이터베이스-구조"></a>
## 6. 데이터베이스 구조

[ERD 링크](https://www.erdcloud.com/d/DMCZQh8JLFqKvuSiX)

<a name="7.-핵심-기능-설명"></a>
## 7. 핵심 기능 설명

<details>
<summary>1. 로그인</summary>

- 토큰 정책
  - 로그인 시 AccessToken 발급, 유효기간 14일
  - 토큰 payload
     ```json
     {
         "userId": "devcamp3",
         "email": "devcamp3@naver.com",
         "jti": "dafbd39f-fe6d-48b5-8661-88230f3803d3",
         "iat": 1684892118,
         "exp": 1686101718
      }
    ```
  >로그인
  <br>

  ![image](https://github.com/DevCamp-TeamSparta/nurseedu-backend/assets/109774037/6b68e36f-523c-41c0-ae03-9348359bf26f)

    <br>

- 다른 브라우저, 기기에서 중복 로그인 시 403에러 발생
- 로그인 시 새로운 JTI가 발급되며, 기존의 토큰은 무효화 됩니다.

   <br>  

</details>

<details>
<summary>2. 수강신청 / 수강이력 기록</summary>

- 용어 설명

| DB 테이블명             | 용도                                                                        |
|------------------|---------------------------------------------------------------------------|
| enrolled         | 수강증입니다. 수강 시작일, 강의 만료일, 강의 상태(결제, 수강중, 수강완료, 만료 등등)을 기록합니다.               |
| course           | 수강할 강의 입니다.                                                               |
| lecure           | 수강할 강의 목차의 데이터 (동영상 link, 동영상 제목 ...)                                     |
| enrolled_lecture | 동영상 시청 이력을 기록하기 위한 메타 테이블 (ex. 시청날짜, 시청 시간, 시청 제한)                        |


+ 수강 신청 내역에서 볼 수 있는 상태 [PENDING]
    + 수강신청하면 course와 1:1 관계인 enrolled 테이블 생성합니다.
    + enrolled의 상태는 PENDING으로 생성합니다. (결제가 이루어지면 INPROGRESS 상태로 변하고 수강할 수 있습니다.)
    + 수강신청 시 course의 자식테이블인 lecture 하나하나에 enrolled_lecture 테이블에 생성합니다.

+ 내 강의실에서 볼 수 있는 상태 [INPROGRESS, COMPLETED]
    + 결제 (enrolled 상태가 PENDING -> INPROGESS로 변경) 강의실에서 수강할 수 있는 상태가 됩니다.
    + 수강 진행 (동영상을 마지막 10초까지 다 수강하면 enrolled_lecture의 is_done이 true로 변경, enrolled의 progress_rate가 백분율로 업데이트)
    + 수강 완료 (enrolled의 progress_rate가 100%가 되면 enrolled 상태가 INPROGRESS -> COMPLETED 변경)

</details>

<details>
<summary>3. 강의 시청시간 제한</summary>

- 강의 playtime의 2배까지 시청 가능

>강의 시청시간 제한
<br>

- 강의 시청은 강의의 playtime(초 단위) 2배로 제한합니다.
- 프론트에서 동영상을 재생하면 1분에 한번씩 서버에 요청을 보내서 DB에 시청 시간(total_seen_time / 초 단위)을 기록합니다. (enrolled_lecture 테이블 / total_seen_time)
- 고객이 동영상을 배속으로 시청하면 배속 만큼 곱셈하여 1분 마다 초 DB에 기록합니다. (ex, 1분 만큼 2배속으로 시청 -> 120초가 enrolled_lecture / total_seen_time에 저장)
- <img width="571" alt="스크린샷 2023-05-25 오전 10 49 03" src="https://github.com/DevCamp-TeamSparta/nurseedu-backend/assets/90877864/c05274bf-3fa5-427a-bdd6-202871f21bfa">
- total_seen_time의 시간이 playtime의 2배를 초과하면 can_watch를 false로 변경합니다.
- 강의 시청시간이 playtime의 2배를 초과하면 서버에서는 409에러를 발생시킵니다.
- 프론트에서는 can_watch를 기준으로 강의 시청을 제한합니다.
- (참고) current_seen_time은 1분짜리 동영상에서 몇초까지 봤다를 표현하기 위해 기록합니다. (enrolled_lecture 테이블 / current_seen_time)

</details>


<a name="8.-API-문서"></a>
##  8. API 문서

- [API 문서 링크](https://api.teamsparta-nurseedue.shop/api-docs#/)


<a name="9.-기타-프로젝트-관련-설명"></a>
## 9. 기타 프로젝트 관련 설명

<details>
<summary>Service와 Repository 분리 방법</summary>

- 코드 가독성을 위해서 Service, Repository를 분리하는 것이 장기적으로 좋습니다.
- TypeORM 0.3.x 이하의 버전에서는 아래의 데코레이터로 Repository를 분리해서 사용했지만, 0.3.x 이후로 해당 데코레이터가 depreciated 되었습니다.

```typescript
@EntityRepository(Entity)
```
- 같은 기능을 하는 CustomDecorator를 만들어서 Repository를 분리했습니다.
- [구현 코드](https://github.com/DevCamp-TeamSparta/nurseedu-backend/commit/15a5ac79e2ea89097a937ce7d9470e976baf1a0f#diff-66a9bdeef87e2c27802e288bbf0c2c0830bd7b2bb21920725e5299beaf2c824a)
- [참고 링크](https://blog.naver.com/pjt3591oo/222927333364)

</details>

<details>
<summary>Swagger 데코레이서 분리해서 1개로 관리하기</summary>

- Swagger 데코레이터를 분리해서 관리하면 가독성이 좋아집니다.
- 각 도메인 폴더에 있는..Decorator.ts를 살펴보면 데코레이터를 변수가 있습니다.
- 해당 변수로 다중의 스웨서 데코레이터를 한번에 선언해 사용할 수 있습니다.

```typescript
// Swagger 데코레이터 모음
export const getAllCourseList = () => {
  return applyDecorators(
    ApiOperation({ summary: '강의 코스 목록 조회(전체)' }),
    ApiOkResponse({
      status: 200,
      description: '강의 코스 목록 조회 성공',
      type: [AllCourseListResDto],
    }),
    ApiNotFoundResponse({
      status: 404,
      description: '강의 코스 목록이 존재하지 않습니다.',
      schema: {
        example: {
          statusCode: 404,
          message: '강의 코스 목록이 존재하지 않습니다.',
          error: 'Not Found',
        },
      },
    }),
  );
};

// 컨트롤러
@ApiTags('COURSE')
@Get(':courseId')
@getCourseDetail() // Swagger 변수
async getCourseDetail(
  @Param('courseId') courseId: number,
): Promise<CourseDetailResDto> {
  return await this.courseService.getCourseDetail(courseId);
}
```

</details>

<details>
<summary>TypeORM Transaction 사용 방법</summary>

- Service Layer에서 Transaction을 걸어야 하는 경우를 위해서 라이브러리를 추가했습니다.
- NestJS의 QueryRunner나 EntityManager를 사용해서 Transaction을 걸 수 있지만, 해당 라이브러리를 사용하면 더 가독성 있게 트랜잭션을 사용할 수 있습니다.
- [typeorm-transactional](https://github.com/Aliheym/typeorm-transactional)
- [참고하면 좋은 글](https://www.timegambit.com/blog/solve/handle-transaction)

</details>

<details>
<summary>Logging 설정</summary>

- 미들웨어를 이용한 로깅 설정
    - 개발 편의를 위해서 모든 요청의 대한 log를 심었습니다.
        - 400 에러는 WARN으로
        - 500 에러는 ERROR로 남겼습니다.
    - ![스크린샷 2023-05-24 오후 5 40 50](https://github.com/DevCamp-TeamSparta/nurseedu-backend/assets/90877864/f8488b5c-cfa4-4614-a175-0649ecc8364e)
    - 개발 편의를 위해서 쿼리의 속도 log를 심었습니다.
    - ![스크린샷 2023-05-24 오후 5 38 56](https://github.com/DevCamp-TeamSparta/nurseedu-backend/assets/90877864/3edd5d0d-f421-4f77-8a7c-986b69c1834c)
    - [코드](https://github.com/DevCamp-TeamSparta/nurseedu-backend/commit/6e93be978d0bd10ceddfe9d30925122d9d4f4124)
    - [참고](https://docs.nestjs.com/middleware)

- TypeORM을 이용한 Logging 설정
    - src/database/DatabaseLogger.ts 파일에 Logging format 설정
    - src/app.module.ts 파일에 DatabaseLogger를 사용하도록 설정
    - [참고](https://docs.nestjs.com/recipes/terminus#logging)

```typescript
@Module({
imports: [
TypeOrmModule.forRootAsync({
  useFactory() {
    return {
    synchronize: postgres.synchronize, // Entity 자동으로 DB 동기화 할지 설정
    logger: new DatabaseLogger(), // TypeORM Logger 설정
    namingStrategy: new ConstraintSnakeNamingStrategy(), // TypeORM Entity의 Column명을 snake_case로 변경
    ssl: {
      rejectUnauthorized: false,
    },
    connectTimeoutMS: postgres.connectTimeout, // DB Connection timeout
    extra: {
      statement_timeout: postgres.queryTimeout, // DB query timeout
    },
  };
},
... 생략
export class AppModule {};
```

</details>

<details>
<summary>js-Joda (시간 라이브러리) 사용시 주의사항</summary>

- JS Date().now 를 사용하면 Zone이 적용이 되지 않아서 실시간을 반영하지 못하는 문제가 있었습니다.
- js-joda 라이브러리를 사용하여 해결 [참고](https://jojoldu.tistory.com/600)
- common/util/util-time에 관련 기능을 구현했습니다.
    - LocalDataTimeTransformer.ts : Entity <-> DB 통신할때 Date <-> LocalDateTime 변환 역할
        - ```typescript
        @Column({
          type: 'timestamp',
          transformer: new LocalDateTimeTransformer(), // 엔티티에 선언 필수 
          nullable: false,
          name: 'updated_at',
        })
        updatedAt: LocalDateTime;
      ```
    - DateTimeUtil.ts : LocalDateTime <-> String, Date 변환 유틸 함수 모음
- 주의사항
    - createQueryBuiler에 LocalDataTime 사용시 Data로 변환이 필요합니다.
        - ClassSerializerInterceptor + Dto 변환시 LocalDataTime을 toString으로 변경해야합니다. 본 프로젝트에서는 ClassSerializerInterceptor를 사용하고 있지 않습니다.

</details>
