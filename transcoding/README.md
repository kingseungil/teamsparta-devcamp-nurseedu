# 널스 에듀 트랜스코딩 서버

## 0. 목차

1. [프로젝트 개요](#1.-프로젝트-개요)
2. [프로젝트의 범위](#2.-프로젝트의-범위)
3. [기술 스택](#3.-기술-스택)
4. [프로젝트 설치/실행](#4.-프로젝트-설치/실행)
5. [인프라 구조](#5.-인프라-구조)
6. [핵심 기능 설명](#6.-핵심-기능-설명)
7. [기타 프로젝트 관련 설명](#7.-기타-프로젝트-관련-설명)

<a name="1.-프로젝트-개요"></a>

## 1. 프로젝트 개요

본 서버는 널스에듀의 온라인강의 트랜스코딩을 위해 개발되었습니다.<br>
백오피스로 업로드한 동영상 파일을 트랜스코딩하여 강의를 시청할 수 있는 형태로 변환하는 서버입니다.

<a name="2.-프로젝트의-범위"></a>

## 2. 프로젝트의 범위

> SQS를 이용한 트랜스코딩 요청

- 백오피스에서 업로드한 동영상 파일을 S3에 업로드
- S3에 업로드된 파일의 정보를 SQS에 전송
- SQS에 전송된 정보를 토대로 트랜스코딩 파이프라인 실행

> 트랜스코딩 파이프라인 구축

- S3에 업로드된 원본 파일을 EFS에 저장
- EFS에 저장된 파일을 압축하여 S3에 업로드, EFS에 저장
- EFS에 저장된 압축 파일을 HLS로 변환 (다수의 ts파일과 m3u8파일로 변환)
- 변환된 파일을 S3에 업로드
- EFS에 저장되어있는 파일을 삭제
- 트랜스코딩 완료 후 SQS 메시지,ECS Task 삭제

> 트랜스코딩 파이프라인 관리

- 트랜스코딩 파이프라인의 상태를 RDS에 저장
- 백오피스에서 트랜스코딩 상태를 확인할 수 있도록 API 제공

<a name="3.-기술-스택"></a>

## 3. 기술 스택

- 언어

| 언어       | 버전    |
| ---------- | ------- |
| Typescript | 4.9.5   |
| NodeJS     | 18.16.0 |

- 프레임워크

| 프레임워크 | 버전  |
| ---------- | ----- |
| NestJS     | 9.0.0 |

- 라이브러리

| 라이브러리                | 버전   | 설명                                       |
| ------------------------- | ------ | ------------------------------------------ |
| fluent-ffmpeg             | 2.1.2  | ffmpeg을 사용하기 쉽게 래핑한 라이브러리   |
| ffmpeg-installer/ffmpeg   | 1.1.0  | ffmpeg을 사용하기 위한 라이브러리          |
| ffprobe-installer/ffprobe | 1.1.0  | ffprobe를 사용하기 위한 라이브러리         |
| @nestjs/cache-manager     | 1.0.0  | NestJS에서 캐시를 사용하기 위한 라이브러리 |
| sqs-consumer              | 5.7.0  | SQS를 사용하기 위한 라이브러리             |
| typeorm                   | 0.3.16 | ORM을 사용하기 위한 라이브러리             |
| pg                        | 8.11.0 | Postgres를 사용하기 위한 라이브러리        |
| js-joda/core              | 5.5.3  | 날짜를 사용하기 위한 라이브러리            |

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

3. 도커로 Redis 실행

```bash
docker run --name redis -p 6379:6379 -d redis
```

4. 로컬에서 프로젝트 실행 (터미널, cmd)

```bash
yarn dev
```

5. 환경별 환경변수 설정

- 환경변수는 .env파일이 아니라 yaml 파일로 관리합니다.

    <details>
    <summary> 1. 설정 파일 작성 </summary>

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
    <summary> 2. yaml 파일에 환경변수 작성 </summary>

    - yaml 파일은 ./config/config.{env}.yaml에 위치

      </details>

      <details>
      <summary> 3. 환경변수 사용 </summary>

        - 프로젝트 내에서 yaml에 작성한 변수가 필요할때 아래와 같이 사용하면 됩니다.

      ```typescript
      const {
        http: { port },
      } = configuration();
      ```

      </details>

      <details>
      <summary> 4. AWS </summary>

        - appspec.\*.yaml : AWS CodeDeploy 설정 파일
        - buildspec.\*.yml : AWS CodeBuild 설정 파일
        - taskdef.\*.json : AWS ECS Task Definition 설정 파일

      </details>

      <details>
      <summary> 5. Docker </summary>

        - proxy/nginx-\*.conf : nginx 설정 파일
        - \*.Dockerfile : 포트 변경하여 사용 가능 (EXPOSE 포트 변경)
        - create-network.sh : Docker network 생성 스크립트 (서비스 이름에 맞게 설정)
        - docker-compose-local.yaml : 서비스 이름, 포트 변경하여 사용 가능
        - Makefile : Docker 명령어 스크립트 (서비스 이름에 맞게 설정)
          > 포트 변경 > 8383 으로 전체검색 후 일괄 변경
          </details>

<a name="5.-인프라-구조"></a>

## 5. 파이프라인 구조

<img width="1677" alt="image" src="https://github.com/DevCamp-TeamSparta/nurseedu-backend/assets/109774037/4d224d13-0886-4b79-b0c3-c6db58ea8583">

<a name="7.-핵심-기능-설명"></a>

## 6. 핵심 기능 설명

<details>
<summary>1. SQS Consumer </summary>

> SQS Consumer는 SQS에 메시지가 들어오면 해당 메시지를 받아서 처리하는 역할을 합니다.

- 기존 ssut/nestjs-sqs 라이브러리를 @Controller 형태로 쓰기위해 내부 패키지 제작 (src/lib/sqs/...)
    - @SqsController(queueName, prefix) : SQS Consumer를 위한 컨트롤러를 생성합니다.
    - @SqsHandler(eventName) : SQS Consumer의 메시지 핸들러를 생성합니다.
- SQS 메시지

    - SQS 메시지는 아래와 같은 형태로 구성됩니다.

  ```typescript
  type PipelineMessage = {
    courseKey: string; // 강의의 고유 키
    fileExtension: string; // 강의의 확장자
    lectureId: number; // 강의의 고유 아이디
    lectureVersion: number; // 강의의 버전
    containerId: string; // ECS 컨테이너의 아이디
  };
  ```

    - 위의 내용을 통해 S3 객체 경로를 구성할 수 있습니다.

  ```typescript
  `${courseKey}/lectures/${lectureId}/video/hls/${lectureVersion}/enc`;
  ```

    - 또한 redis에 트랜스코딩 상태를 저장할 때 사용할 수 있습니다.

  ```typescript
  this.redisAdapter.set(`${lectureId}`, value);
  ```

    - value는 아래와 같은 형태로 구성됩니다.
      ```typescript
      type PipelineStatusForCache = {
        containerId: string; // ECS 컨테이너의 아이디
        status: PipelineStatus; // 트랜스코딩 상태
        convertProgress?: string | number; // 변환 진행률
        compressProgress?: string | number; // 압축 진행률
        transcodeProgress?: string | number; // 트랜스코딩 진행률
      };
      ```
        - PipelineStatus는 아래와 같은 형태로 구성됩니다.
          ```typescript
          PENDING: 'PENDING', // 대기
          START: 'START', // 시작
          DOWNLOADING: 'DOWNLOADING', // 다운로드 중
          CONVERTING: 'CONVERTING', // 변환 중
          CONVERTED_UPLOADING: 'CONVERTED_UPLOADING', // 변환 완료 후 업로드 중
          COMPRESSING: 'COMPRESSING', // 압축 중
          COMPRESS_UPLOADING: 'COMPRESS_UPLOADING', // 압축 완료 후 업로드 중
          TRANSCODING: 'TRANSCODING', // 트랜스코딩 중
          TRANSCODE_UPLOADING: 'TRANSCODE_UPLOADING', // 트랜스코딩 완료 후 업로드 중
          COMPLETED: 'COMPLETED', // 완료
          FAILED: 'FAILED' // 실패
          ```

- SQS Consumer 사용법
    - 백오피스에서 'pipeline' 이라는 sqs 컨트롤러에 'execute' 라는 이벤트를 발생시키면 트랜스코딩 서버의 SqsConsumer가 해당 메시지를 받아서 처리합니다.
    - SqsConsumer는 해당 메시지를 받아서 PipelineService를 통해 트랜스코딩을 진행합니다.
  ```typescript
  @SqsController(sqs.name, 'pipeline')
  @Injectable()
  export class SqsPipelineConsumer {
    @SqsHandler('execute')
    async handleTranscodingMessage(message: any) {
      const pipelineParam: PipelineMessage = { ...message[0] };
      const result = await this.pipelineService.startPipeline(pipelineParam);
      // ...
    }
  }
  ```

</details>

<details>
<summary>2. Transcoding Pipeline </summary>

> **동작 과정** <br>
>
> - PipelineService는 PipelineDomainEntity를 통해 Pipeline의 상태를 관리합니다.
> - 파이프라인 동작 과정은 아래와 같습니다.
    >   - S3에 업로드된 파일을 efs에 다운로드 합니다.
>   - 다운로드된 파일이 mp4 파일이 아니면, ffmpeg을 통해 mp4 파일로 변환합니다.
>   - 변환된 mp4 파일을 S3에 업로드 합니다.
>   - mp4 파일을 압축합니다.
>   - 압축된 파일을 S3에 업로드 합니다.
>   - mp4 파일을 트랜스코딩 합니다.
>   - 트랜스코딩된 파일을 S3에 업로드 합니다.
> - S3에 업로드된 m3u8 파일을 플레이어에서 재생할 수 있도록 DB lecture 테이블에 저장합니다.
    >   - accessKey는 플레이어에서 m3u8 파일을 재생할 때 필요한 키입니다.
> - 위 과정까지 끝나면, efs에 저장된 파일을 삭제하고 consumer에 'success'를 반환합니다.

```typescript
public async startPipeline(message: PipelineMessage): Promise<string> {
  console.log('Start Pipeline');
  const { containerId } = message;
  const domainEntity = new PipelineDomainEntity({
    status: PipelineStatus.START,
    containerId,
  });

  await this.download(message, domainEntity);

  // If we need convert video, do convert
  if (message.fileExtension !== 'mp4') {
    await this.convert(message, domainEntity);
    await this.upload(message, domainEntity);
  }
  await this.compress(message, domainEntity);
  await this.upload(message, domainEntity); // upload compressed file
  await this.transcode(message, domainEntity);
  await this.upload(message, domainEntity); // upload transcoded file

  // Update status to completed
  domainEntity.setStatus(PipelineStatus.COMPLETED);
  await this.updatePipelineStatus(message, domainEntity);
  const accessKey = 'nurseeduNURSEEDUcamp';
  await this.postURL(message.lectureId, accessKey);
  await this.efsManagerAdapter.deleteAllFilesAndFolders(message);
  return Promise.resolve('success');
}
```

</details>

<details>
<summary>3. Compress </summary>

> **영상 압축 로직** <br>
>
> - 압축을 통해 저장 공간을 절약하고, 트랜스코딩 시간을 단축시킵니다.
> - ffmpeg 라이브러리를 사용하여 압축을 진행합니다.
>
> **동작 과정** <br>
>
> - 입력 동영상의 메타데이터(FfprobeData 형식)를 추출합니다. (getMetaData 함수)
> - 메타데이터를 바탕으로 압축할 동영상의 비트레이트를 선택합니다. (selectBitrate 함수)
> - 선택된 비트레이트를 바탕으로 ffmpeg 라이브러리를 사용하여 압축을 진행합니다. (compressMp4 함수)
>
> **코드 설명** <br>
>
> - compressMp4 : 동영상을 압축하는 주요 함수입니다. 인자로 inputPath, outputPath, options를 받아서 압축 작업을 수행하고 압축한 동영상의 경로를 반환합니다.
    >   - ffmpeg 압축 옵션
          >     - crf : 압축률을 나타내는 값입니다. 0 ~ 51의 값을 가질 수 있으며, 0에 가까울수록 압축률이 높습니다. 기본값은 23입니다.
>     - preset : 압축 방식을 나타내는 값입니다. slow, medium, fast, veryfast, superfast, ultrafast 중 하나를 선택할 수 있습니다. 기본값은 medium입니다.
>     - c:v (비디오 코덱) : libx264
>     - c:a (오디오 코덱) : aac
>   - 압축 과정에서 발생하는 이벤트
      >     - codecData : 코덱 데이터
>     - start : 압축 시작
>     - progress : 압축 진행률
>     - end : 압축 종료
>     - error : 압축 에러
> - getMetaData : 입력 동영상의 메타데이터를 추출하는 함수입니다. 인자로 동영상 파일의 경로를 받아서 메타데이터를 추출합니다.
> - selectBitrate : 입력 동영상의 메타데이터를 바탕으로 압축할 동영상의 비트레이트를 선택하는 함수입니다. 인자로 동영상의 크기를 받아서 비트레이트를 선택합니다.

</details>

<details> 
<summary>4. Transcode to HLS </summary>

> **HLS 변환 로직** <br>
>
> - MP4 파일을 HLS 형식으로 트랜스코딩하는 로직입니다.
> - ffmpeg 라이브러리를 사용하여 트랜스코딩을 진행합니다.
>
> **동작 과정** <br>
>
> - inputPath, outputPath, options를 인자로 받습니다.
> - hlsOption을 설정하고 필요한 경우 암호화 옵션도 추가합니다.
> - ffmpeg 라이브러리를 사용하여 영상을 여러 해상도의 동영상 세그먼트로 분할합니다.
> - 분할된 동영상을 m3u8 파일로 묶습니다.
> - 트랜스코딩이 완료되면 outputPath를 반환합니다.
>
> **코드 설명** <br>
>
> - transcodeMp4ToHlsWithFfmpeg: MP4 동영상 파일을 HLS 형식으로 트랜스코딩하는 주요 함수입니다. 인자로 inputPath, outputPath, option을 받아서 트랜스코딩 작업을 수행하고 트랜스코딩된 동영상의 경로를 반환합니다.
    >   - 구성 요소와 기능에 대한 간단한 단계별 설명은 위의 "동작 과정"을 참조하십시오.
> - hlsOption: HLS 트랜스코딩에 필요한 옵션을 설정하는 함수입니다.
    >   - preset : 트랜스코딩에 사용할 프리셋을 설정합니다. (인코딩 속도)
>   - hls_list_size : HLS 목록 개수를 설정합니다. (0으로 설정하면 모든 세그먼트를 포함합니다.)
>   - threads : 코어당 스레드 개수를 설정합니다. (0으로 설정하면 코어 개수만큼 스레드를 생성합니다.)
>   - f : 트랜스코딩 결과물 포맷을 설정합니다. (hls)
>   - hls_time : 개별 파일 길이를 설정합니다. (seconds)
>   - hls_flags : 트랜스코딩에 사용할 플래그를 설정합니다. (delete_segments, round_durations, independent_segments...)
> - ffmpeg 트랜스코딩 옵션
    >   - outputOptions : 압축 옵션과 항목이 동일합니다. + hlsOption
>   - outputOption : 4개의 출력을 생성합니다.
      >     - 0번째 결과 : 640x360 해상도와 600k 비디오 비트레이트, 1000k 오디오 비트레이트를 가집니다
>     - 1번째 결과 : 960x540 해상도와 1500k 비디오 비트레이트, 1000k 오디오 비트레이트를 가집니다
>     - 2번째 결과 : 1280x720 해상도와 3000k 비디오 비트레이트, 1000k 오디오 비트레이트를 가집니다
>     - 3번째 결과 : 1920x1080 해상도와 5000k 비디오 비트레이트, 1000k 오디오 비트레이트를 가집니다
>     - 결과물을 -%v.m3u8 형식으로 저장합니다.
> - 트랜스코딩 과정에서 발생하는 이벤트
    >   - 압축 로직과 유사합니다.
>   - 'end' 이벤트에서 manifest 파일을 생성합니다.
      >     - manifest 파일은 hls 파일의 목록과 정보를 담고 있는 파일입니다.
>     - manifest 파일의 내용은 hlsOption에 따라 달라집니다.

</details>

## 7. 기타 프로젝트 관련 설명

<details> 
<summary>...</summary>

</details>
