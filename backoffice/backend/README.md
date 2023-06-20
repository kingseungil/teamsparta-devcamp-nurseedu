# 널스 에듀 백오피스 서버

## 0. 목차

- [널스 에듀 백오피스 서버](#널스-에듀-백오피스-서버)
    - [0. 목차](#0-목차)
    - [1. 프로젝트 개요](#1-프로젝트-개요)
    - [2. 프로젝트의 범위](#2-프로젝트의-범위)
    - [3. 기술 스택](#3-기술-스택)
    - [4. 프로젝트 설치/실행](#4-프로젝트-설치실행)
        - [redis docker](#redis-docker)
        - [Installation](#installation)
    - [5. 인프라 구조](#5-인프라-구조)
    - [6. 데이터베이스 구조](#6-데이터베이스-구조)
    - [7. 핵심 기능 설명](#7-핵심-기능-설명)
        - [7-2. API 명세서](#7-2-api-명세서)
        - [기타](#기타)

<a name="1.-프로젝트-개요"></a>

## 1. 프로젝트 개요

본 서버는 널스에듀 백오피스로 서비스 관리를 위해 만들어졌습니다.<br/>
서비스에 제공할 강의 및 영상 추가, 수정 및 업로드를 할 수 있습니다.

<a name="2.-프로젝트의-범위"></a>

## 2. 프로젝트의 범위

<details>
<summary>AUTH</summary>
<div markdown='1'></div>
<br/>

- 관리자는 '**로그인**' 할 수 있다.
</details>

<details>
<summary>ADMIN</summary>
<div markdown='1'></div>
<br/>

- 관리자는 '**유저 목록**'을 조회할 수 있다.
- 관리자는 '**유저 상세 조회**'를 할 수 있다.
- 관리자는 '**유저 정보 변경**'을 할 수 있다.
- 관리자는 '**유저 삭제**'를 할 수 있다.
- 관리자는 '**새로운 관리자**'를 등록할 수 있다.
- 관리자는 '**새로운 튜터**'를 등록할 수 있다.
</details>

<details>
<summary>COURSE</summary>
<div markdown='1'></div>
<br/>

- 관리자는 '**강의 목록**'을 조회할 수 있다.
- 관리자는 '**강의 상세 조회**'를 할 수 있다.
- 관리자는 '**새로운 강의**'를 등록할 수 있다.
- 관리자는 '**강의를 수정**'할 수 있다.
- 관리자는 '**강의를 삭제**'할 수 있다.
- 관리자는 '**강의를 등록 완료 상태**'로 변경할 수 있다.
    - 영상이 업로드된 수업이 하나 이상 존재해야 합니다.
    - 등록 완료 상태가되어야 서비스에 노출됩니다.
- 관리자는 '**강의를 등록 대기 상태**'로 변경할 수 있다.
</details>

<details>
<summary>LECTURE</summary>
<div markdown='1'></div>
<br/>

- 관리자는 '**수업 목록**'을 조회할 수 있다.
- 관리자는 '**수업 상세 조회**'를 할 수 있다.
- 관리자는 '**새로운 수업**'를 등록할 수 있다.
- 관리자는 '**수업을 수정**'할 수 있다.
- 관리자는 '**수업 순서 정렬**'을 할 수 있다.
- 관리자는 '**수업을 삭제**'할 수 있다.
- 관리자는 '**수업을 등록 완료 상태**'로 변경할 수 있다.
    - 영상 트랜스코딩 완료시 등록 완료상태로 변경 됩니다.
- 관리자는 '**수업을 등록 대기 상태**'로 변경할 수 있다.
</details>

<details>
<summary>CATEGORY</summary>
<div markdown='1'></div>
<br/>

- 관리자는 '**카테고리 목록**'을 조회할 수 있다.
- 관리자는 '**새로운 카테고리**'를 등록할 수 있다.
- 관리자는 '**카테고리 수정**' 할 수 있다.
- 관리자는 '**카테고리 삭제**' 할 수 있다.
    - 카테고리에 속한 강의가 존재할 시 삭제할 수 없습니다.
  </details>

<details>
<summary>TAG</summary>
<div markdown='1'></div>
<br/>

- 관리자는 '**태그 목록**'을 조회할 수 있다.
- 관리자는 '**새로운 태그**'를 등록할 수 있다.
- 관리자는 '**태그 수정**' 할 수 있다.
- 관리자는 '**태그 삭제**' 할 수 있다.
    - 태그에 속한 강의가 존재할 시 삭제할 수 없습니다.
  </details>

<details>
<summary>ENROLLED</summary>
<div markdown='1'></div>
<br/>

- 관리자는 유저의 '**수강 목록**'을 조회할 수 있다.
- 관리자는 유저의 '**수강 정보**'를 수정할 수 있다.
- 관리자는 유저의 '**수강 정보**'를 삭제할 수 있다.
- 관리자는 유저에게 '**강의 지급**'을 할 수 있다.
</details>

<details>
<summary>ORDER</summary>
<div markdown='1'></div>
<br/>

- 관리자는 유저의 '**주문 목록**'을 조회할 수 있다.
- 관리자는 유저의 '**주문 정보**'를 수정할 수 있다.
- 관리자는 유저의 '**주문 정보**'를 삭제할 수 있다.
</details>

<a name="3.-기술-스택"></a>

## 3. 기술 스택

- 언어

| 언어       | 버전    |
| ---------- | ------- |
| Typescript | 4.7.4   |
| NodeJS     | 18.16.0 |

- 프레임워크

| 프레임워크 | 버전  |
| ---------- | ----- |
| NestJS     | 9.0.0 |

- 데이터베이스 관련

| 데이터베이스        | 버전   |
| ------------------- | ------ |
| Amazon RDS Postgres | 15.2   |
| TypeORM             | 0.3.15 |

- 라이브러리

| 라이브러리                   | 버전  |                                                                                                                       |
| ---------------------------- | ----- | --------------------------------------------------------------------------------------------------------------------- |
| passport                     | 0.6.0 | 로그인 관련 라이브러리                                                                                                |
| passport-jwt                 | 4.0.1 | 로그인 관련 라이브러리                                                                                                |
| @js-joda/core                | 5.5.3 | [시간 관련 라이브러리](https://jojoldu.tistory.com/600)                                                               |
| @types/js-yaml               | 4.0.5 | 환경변수 yaml로 관리할 수 있도록 도와주는 라이브러리                                                                  |
| js-yaml                      | 4.1.0 | 환경변수 yaml로 관리할 수 있도록 도와주는 라이브러리                                                                  |
| typeorm-transactional        | 0.4.1 | TypeORM 0.3 사용시 Service 계층에서 트랜잭션을 걸어주는 역할 [링크](https://github.com/Aliheym/typeorm-transactional) |
| @nestjs/cache-manager        | 1.0.0 | NestJS 캐시 관련 라이브러리                                                                                           |
| cache-manager                | 5.2.1 | Node.js 캐시 관련 라이브러리                                                                                          |
| @types/cache-manager         | 4.0.2 | Node.js 캐시 관련 라이브러리                                                                                          |
| cache-manager-ioredis        | 2.1.0 | 캐시 저장소를 Redis를 사용                                                                                            |
| @types/cache-manager-ioredis | 2.0.3 | 캐시 저장소를 Redis를 사용                                                                                            |

<a name="4.-프로젝트-설치/실행"></a>

## 4. 프로젝트 설치/실행

1. 프로젝트 클론

### redis docker

```shell
docker run --rm -it -p 6379:6379 --name redis -d redis
```

### Installation

```bash
git clone {주소}
```

1. 패키지 설치

```bash
yarn install
```

3. 환경변수 설정

- yaml 파일은 프로젝트 최상단 config 폴더에 위치해있습니다.

```bash
env: local

service:
  name: 'nurseedu-backoffice-backend'

http:
  host: 'localhost'
  port: 3000

sqs:
  name: ''
  url: ''
  region: 'ap-northeast-2'

aws: # aws 설정
  accessKeyId: '' # accessKeyId
  secretAccessKey: '' #secretAccessKey
  region: 'ap-northeast-2' #region

s3: # S3 설정
  bucket: ''
  region: 'ap-northeast-2'
  credentials:
    accessKeyId: ''
    secretAccessKey: ''

ecs: # task 설정
  clusterName: 'transcoding'
  credentials:
    accessKeyId: ''
    secretAccessKey: ''
  region: 'ap-northeast-2'
  taskDefinition: ''
  networkConfiguration:
    awsvpcConfiguration:
      subnets:
        - ''
        - ''
      securityGroups:
        - ''
      assignPublicIp: 'ENABLED'
```

4. 로컬에서 프로젝트 실행 (터미널, cmd)

```bash
yarn dev
# 특이사항 (yaml 파일로 환경설정을 위해서 위 명령어로 아래의 스크립트가 동작한다. package.json 파일 위치)
"dev": "rm -rf dist/config/config.*.yaml && mkdir -p dist/config && cp config/config.local.yaml dist/config && cross-env ENV=local nest start --watch"
```

<a name="5.-인프라-구조"></a>

## 5. 인프라 구조

- 인프라
  ![nurseedu-backoffice](http://github.com/DevCamp-TeamSparta/nurseedu-backoffice-backend/assets/117348491/862d888c-b503-438c-ad21-ba36f8b4e0ca)

<a name="6.-데이터베이스-구조"></a>

## 6. 데이터베이스 구조

![nurseedu](https://github.com/DevCamp-TeamSparta/nurseedu-backoffice-backend/assets/117348491/188e9294-99ea-4bb3-9a24-3c81bea4f8fa)

[링크](https://www.erdcloud.com/d/TckXtoKunBRuXTrX7)

<a name="7.-핵심-기능-설명"></a>

## 7. 핵심 기능 설명

<details>
 <summary>강의 등록</summary>
 <div markdown='1'>

- POST /courses

- 요청 헤더
    - Authorization: Bearer 토큰 형식의 사용자 인증 토큰

- 요청 본문

    ```json
      {
        "title": "string",      // 강의 타이틀
        "courseKey": "string",  // s3 업로드 키
        "days": number,         // 수강 기간
        "price": number,        // 가격
        "categoryIds": [number, number, ...],
        "teacherIds": [number, number, ...],
        "tagIds": [number, number, ...],  // optional
        "textbook": "string"  // optional
      }
    ```

- 응답 본문

  ```js
    // 요청 헤더에 토큰이 없을시
    {
      "statusCode": 401,
      "message": "Unauthorized"
    }
    // title 중복시
    {
      "statusCode": 400,
      "message": "중복 강의명은 허용하지 않습니다."
    }
    // courseKey 중복시
    {
      "statusCode": 400,
      "message": "중복된 courseKey가 존재합니다."
    }
    // 카테고리가 존재하지 않을 시
    {
      "statusCode": 404,
      "message": "id 와 일치하는 카테고리가 없습니다."
    }
    // 튜터가 존재하지 않을 시
    {
      "statusCode": 404,
      "message": "id 와 일치하는 튜터가 존재하지 않습니다."
    }
    // 태그가 존재하지 않을 시
    {
      "statusCode": 404,
      "message": "id 와 일치하는 태그가 없습니다."
    }
  ```

- 동작

    1. 요청 헤더에서 토큰을 추출하고 검증합니다.
    2. 요청 본문에서 강의 정보를 받아와 유효성 검사를 합니다.
        1. courseKey 중복 검사를 합니다.
        2. title 중복 검사를 합니다.
        3. category가 존재하지는지 확인합니다.
        4. teacher가 존재하는지 확인합니다.
        5. tag가 존재하는지 확인합니다.
    3. 강의를 등록합니다.
       </div>
       </details>
  <details>
  <summary> 강의 수정</summary>
  <div markdown='1'>

    - PUT /courses/:courseId

    - 요청 헤더

        - Authorization: Bearer 토큰 형식의 사용자 인증 토큰

    - 요청 본문

  ```json
    {
      "title": "string",      // 강의 타이틀  optional
      "courseKey": "string",  // s3 업로드 키  optional
      "days": number,         // 수강 기간  optional
      "price": number,        // 가격  optional
      "categoryIds": [number, number, ...], // optional
      "teacherIds": [number, number, ...], // optional
      "tagIds": [number, number, ...],  // optional
      "textbook": "string"  // optional
    }
  ```

    - 응답 본문

  ```js
    // 요청 헤더에 토큰이 없을시
    {
      "statusCode": 401,
      "message": "Unauthorized"
    }
    // title 중복시
    {
      "statusCode": 400,
      "message": "중복 강의명은 허용하지 않습니다."
    }
    // 카테고리가 존재하지 않을 시
    {
      "statusCode": 404,
      "message": "id 와 일치하는 카테고리가 없습니다."
    }
    // 튜터가 존재하지 않을 시
    {
      "statusCode": 404,
      "message": "id 와 일치하는 튜터가 존재하지 않습니다."
    }
    // 태그가 존재하지 않을 시
    {
      "statusCode": 404,
      "message": "id 와 일치하는 태그가 없습니다."
    }
  ```

    - 동작

    1. 요청 헤더에서 토큰을 추출하고 검증합니다.
    2. 요청 본문에서 강의 정보를 받아와 유효성 검사를 합니다.
        1. title 중복 검사를 합니다.
        2. category가 존재하지는지 확인합니다.
        3. teacher가 존재하는지 확인합니다.
        4. tag가 존재하는지 확인합니다.
    3. 강의를 수정합니다.
  </div>
  </details>

    <details>
    <summary>수업 영상 업로드</summary>
    <div markdown='1'>

    - GET /s3/upload/:lectureId

    - 요청 헤더

        - Authorization: Bearer 토큰 형식의 사용자 인증 토큰

    - 요청 쿼리

  ```json
    "s3DirPath" = `${courseKey}/lectures/${order}/video/${fileExtension}/${version}/original`
    "filename" = filename.(mp4 | m4v | hls)
    "ext" = mp4 or m4v ... etc
  ```

    - 응답 본문

  ```js
   https://s3.ap-northeast-2.amazonaws.com/course-materials.nurse-edu.co.kr/s3DirPath/filename?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA4GXI47Q7VLHW4DGO%2F20230523%2Fap-northeast-2%2Fs3%2Faws4_request&X-Amz-Date=20230523T043304Z&X-Amz-Expires=7200&X-Amz-Signature=50c7ec7e002b90a8f56bf8464ee74011e3e925a21994390c3527843f7e8d218d&X-Amz-SignedHeaders=host

   // 프론트에서 응답 Url로 PUT 요청시 s3에 업로드할 수 있습니다.
  ```

    - 동작

        1. 요청 헤더에서 토큰을 추출하고 검증합니다.
        2. path.join(s3DirPath, filename) 으로 s3내의 Key(저장위치)와 Object이름을 결정합니다.
        3. ext를 Lecture 테이블에 origin_file_extension 컬럼에 저장합니다.
        4. 생성된 upload Url을 반환합니다.
      </div>
      </details>

      <details>
      <summary>수업 링크 업로드</summary>
      <div markdown='1'>

        - POST /courses/lectures/url

        - 요청 헤더

            - X-nurseedu-access-key # 트랜스코딩을 위한 검증 키

        - 요청 본문

          ```json
            "lectureId": number
          ```

        - 응답 본문

          ```json
            // header에 X-nurseedu-access-key 가 없을시
            {
              "statusCode": 403,
              "message": "접근 권한이 없습니다."
            }
            // X-nurseedu-access-key 의 값이 올바르지 않을시
            {
              "statusCode": 403,
              "message": "접근 권한이 없습니다."
            }
          ```

        - 동작
            - 트랜스코딩이 완료되면 해당 api를 호출합니다.
            - header의 X-nurseedu-access-key를 검증합니다.
            - body의 lectureId로 lecture를 조회합니다.
            - 조회한 lecture로 link (트랜스코딩이 완료된 .m3u8) 를 생성합니다.
            - 생성한 link를 Lecture 테이블의 link 컬럼에 저장합니다.

      </div>
      </details>

      <details>
      <summary>트랜스코딩 </summary>
      <div markdown='1'>

        - src/transcoding/TranscodingService.ts

        - GET /transcode/start

        - 요청 헤더

            - Authorization: Bearer 토큰 형식의 사용자 인증 토큰

        - 요청 쿼리

          ```json
            containerId: string,
            fileExtension: string
          ```

        - 응답 본문

          ```json
            //maybe 추후 수정
            {
              "courseKey": string,
              "lectureOrder": number,
              "fileExtension": string,
              "lectureId": number,
              "lectureVersion": number,
              "containerId": string
            }
          ```

        - 동작
            - ECS 태스크 컨테이너를 실행합니다.# nurseedu-transcoding-prod-td
            - pipeline 실행에 필요한 message를 만들고 전송합니다.
            - 트랜스코딩 시작 상태를 레디스에 저장합니다.

      </div>
      </details>

      <details>
      <summary>트랜스코딩 상태 확인</summary>
      <div markdown='1'>

        - src/transcoding/TranscodingService.ts
        - 프론트에서 트랜스코딩 시작시 5초마다 해당 API를 통해 상태를 확인합니다.
        - 트랜스코딩 서버에서 Redis에 상태와 진행율을 저장합니다.

            - src/transcoding-pipeline/application/services/vod/vod.service.ts:157
            - src/transcoding-pipeline/application/services/pipeline/pipeline.service.ts: 87

        - GET /transcode/status/:lectureId

        - 요청 헤더

            - Authorization: Bearer 토큰 형식의 사용자 인증 토큰

        - 요청 파라미터

          ```json
            lectureId: string
          ```

        - 응답 본문

          ```json
            //maybe 추후 수정
              {
                "containerId": containerId,
                "status": status,
                "convertProgress": convertProgress,
                "compressProgress": compressProgress,
                "transcodeProgress": transcodeProgress,
              }
          ```

        - 동작
            - redis에서 트랜스코딩의 현재 작업 상태와 진행도를 읽어옵니다.
            - 읽어온 데이터는 단순 string으로 되어있기에 Object로 Parse를 합니다.
            - 프론트로 parse한 Object를 반환합니다.
          </div>
          </details>

      <!-- 트랜스코딩 서버에서 호출하는 API -->
      <details>
      <summary>수업 링크 업데이트 & 활성화</summary>
      <div markdown='1'>

        - POST courses/lectures/url
        - src/lecture/LectureService.ts: updateUrl()
        - 트랜스코딩 서버에서 트랜스코딩이 완료되면 해당 url을 호출합니다.

        - 요청 헤더

            - X_nurseedu_access_key=nurseeduNURSEEDUcamp
            - 트랜스코딩 서버는 토큰을 가지고 있지 않기에 커스텀헤더를 이요한 검증을 하고 있습니다.
            - src/lecture/decorator/ValidateUpdateReqHeader.ts

        - 요청 본문

          ```json
            lectureId: number
          ```

        - 동작
            - Body의 lectureId로 lecture를 조회합니다
            - 조회한 lecture로 m3u8이 저장된 s3 link를 생성합니다.
            - 생성한 link를 lecture 테이블에 저장합니다.
            - lecture를 isActive = true 로 수정합니다.
            - lectureId로 Enrolled(수강 유저)가 존재하는지 확인합니다.
                - Enrolled(수강 유저)가 존재할 시 해당 Enrolled에 EnrolledLecture를 추가합니다.

      </div>
      </details>

  ### 7-2. API 명세서

    - [API 문서 링크](https://api.backoffice.teamsparta-nurseedue.shop/api-docs)

  ### 기타

    <details>
  <summary>Swagger 데코레이서 분리해서 1개로 관리하기</summary>

- Swagger 데코레이터를 분리해서 관리하면 가독성이 좋아집니다.
- 각 도메인 폴더에 있는..Decorator.ts를 살펴보면 데코레이터를 변수가 있습니다.
- 해당 변수로 다중의 스웨거 데코레이터를 한번에 선언해 사용할 수 있습니다.

```typescript
// Swagger 데코레이터 모음
export const getAllUsersDoc = () => {
  return applyDecorators(
    ApiBearerAuth('accessToken'),
    ApiQuery({
      name: 'sortby',
      description: '정렬 기준',
      required: false,
      type: String,
    }),
    ApiQuery({
      name: 'type',
      description: '조회하고자 하는 유저의 타입 ex) user, teacher, admin',
      required: false,
    }),
    ApiOperation({ summary: '유저 목록 조회' }),
    ApiOkResponse({
      status: 200,
      description: '유저 목록 조회 성공',
      type: UserDto,
      isArray: true,
    }),
    ApiUnauthorizedResponse(unAuthorizedOptions),
  );
};

// 다음과 같이 사용 가능합니다.
@getAllUsersDoc()
@Get()
async getAllUsers(
  @Query('sortby', new DefaultValuePipe('updatedAt')) sortby: string,
  @Query('type', new DefaultValuePipe(''))
  type: string,
): Promise<UserDto[]> {
  return await this.userService.getAllUsers(sortby, type);
}
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
    - createQueryBuilder에 LocalDataTime 사용시 Data로 변환이 필요합니다.
        - ClassSerializerInterceptor + Dto 변환시 LocalDataTime을 toString으로 변경해야합니다. 본 프로젝트에서는 ClassSerializerInterceptor를 사용하고 있지 않습니다.

</details>
