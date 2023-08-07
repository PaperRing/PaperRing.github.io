---
title: "[Spring Boot] S3 PresignedUrl로 이미지 업로드 하기"
date: 2023-04-01 11:33:00 +0800
categories: [AWS]
tags: [SpringBoot, Kotlin, AWS]
pin: true
math: true
mermaid: true
comments: true
toc: true
layout: post

  
---
## **1. PresignedUrl을 사용하는 이유**
---
이미지 업로드는 부하가 큰 작업이라, 백엔드 서버를 거치게 되면 서버가 금방 죽을 수 있다. 혹은 동시 업로드 요청수를 제한해야 하는데 이는 유저경험이 안 좋아질 수 있다는 단점이 있다. 이때 PresignedUrl을 이용하면 백엔드 서버를 거치지 않고 클라이언트에서 S3로 바로 업로드가 가능해진다. 원래는 이미지를 백엔드가 S3로 전달함과 동시에 보안절차도 한 번에 진행되는데, 우리는 이 과정을 분리해서 백엔드에서는 presignedUrl만 생성해서 보안절차 작업만 해줄 것이다. 그러면 클라이언트가 s3로 바로 업로드 가능하다.

<br>
## **2. 버킷 만들기**
---
![](https://blog.kakaocdn.net/dn/YEPmf/btsqocN8F4l/fldjDDinHSe5HO38VF4dy0/img.png)

먼저 버킷 이름을 정해준다.

![](https://blog.kakaocdn.net/dn/b62hq8/btsqj4iwYh2/lM2nhXNGJxlhxdzobWGA30/img.png)

프론트에서 직접 업로드하기 위해서는 퍼블릭 액세스를 허용해줘야 한다.

아래에 있는 설정은 그대로 두고 버킷을 생성해 줌.

![](https://blog.kakaocdn.net/dn/VyaG7/btsp7JGPVjp/gmbZXUtUsr5LnRI2d6aRUK/img.png)

생성된 버킷에 들어가서 권한탭을 눌러준다.

![](https://blog.kakaocdn.net/dn/chrGjU/btsp8zxjRjX/YDWtrAN8xBGlX8C5PnBIqk/img.png)

버킷 정책을 설정해줘야 하는데, 편집에 들어가서

![](https://blog.kakaocdn.net/dn/eAnmVx/btsqh3D2cmI/jmClseBTfWLWeRgLHrn25K/img.png)

정책 생성기를 눌러준다.

![](https://blog.kakaocdn.net/dn/c7t0mi/btsp7U9x0Fw/5VA6xy4yMKc1oJRcUB9jf0/img.png)

Select Type of Policy: S3 Bucket Policy  
Effect: Allow  
Principal: *  
Actions: GetObject, PutObject  
Amazon Resource Name (ARN): 해당 버킷 ARN 입력 (이전 페이지에 있음)

![](https://blog.kakaocdn.net/dn/bkUZDn/btsqrDdD5ef/7uB6Kb0PcySMf2HGKVAkc1/img.png)

Add Statement를 누르면 아래 정책이 생긴 걸 확인할 수 있고, Generate Policy를 클릭한다.

생성된 정책을 복사하고, 이전 페이지로 돌아가 붙여 넣어준다.

```
{
    "Version": "2012-10-17",
    "Id": "Policy1680399902110",
    "Statement": [
        {
            "Sid": "생성된 sid",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:Put*",
                "s3:Get*"
            ],
            "Resource": "arn:aws:s3:::paperring-bucket/*"
        }
    ]
}
```

이때, ARN 주소인 Resource 마지막에 /* 를 추가해 줘야지 public권한이 생성된다.

![](https://blog.kakaocdn.net/dn/bHrypu/btsqtugBMNz/CtZdsrHIYylKk2hdkPvSF0/img.png)

권한 개요에 퍼블릭이라고 떠야지 접근 가능하니 꼭 확인해야 함

![](https://blog.kakaocdn.net/dn/bdcQ6C/btsqfACgp5i/w0B99YTxbiw5I7f6fnRcQ0/img.png)

다음으로 cross origin 설정을 해야 한다.

```
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "HEAD",
            "GET",
            "PUT",
            "POST",
            "DELETE"
        ],
        "AllowedOrigins": [
            "*"
        ],
        "ExposeHeaders": [
            "ETag"
        ]
    }
]
```

위의 설정을 넣어주면 버킷 설정 완료!

<br>
## **3. Spring Boot / Kotlin 설정**
---
```
implementation("org.springframework.cloud:spring-cloud-starter-aws:2.2.6.RELEASE")
```

`build.gradle` 파일에 라이브러리를 추가해 준다.

```
cloud:
  aws:
    credentials:
      accessKey: [access-key]
      secretKey: [secret-key]
    s3:
      bucket: [설정해준 버킷 이름]
    region:
      static: ap-northeast-2
    stack:
      auto: false
```

`application.yml`에 추가

인증키는 IAM에서 확인하면 된다.

```kotlin
@Configuration
class S3Config(
    @Value("\${cloud.aws.credentials.accessKey}")
    private val accessKey: String,

    @Value("\${cloud.aws.credentials.secretKey}")
    private val secretKey: String,

    @Value("\${cloud.aws.region.static}")
    private val region: String
) {


    @Bean
    fun amazonS3Client(): AmazonS3Client {
        val awsCredentials = BasicAWSCredentials(accessKey, secretKey)
        return AmazonS3ClientBuilder.standard()
            .withRegion(region)
            .withCredentials(AWSStaticCredentialsProvider(awsCredentials))
            .build() as AmazonS3Client
    }
}
```

S3 Config 파일을 생성해 위와 같이 설정해 준다.

```kotlin
@Service
@Component
class S3Service(
    private val s3Config: S3Config,
    @Value("\${cloud.aws.s3.bucket}")
    private val bucket: String
) {

    fun getPreSignedUrl(folder: String, fileName: String): String {
        val encodedFileName = "${fileName}"
//      s3에 실제 저장될 경로
        val objectKey = "${folder}/${encodedFileName}"

        val expiration = Date()
        var expTimeMillis: Long = expiration.time
        expTimeMillis += (3 * 60 * 1000).toLong() // 3분
        expiration.time = expTimeMillis // url 만료 시간 설정

        val generatePresignedUrlRequest: GeneratePresignedUrlRequest =
            GeneratePresignedUrlRequest(bucket, objectKey)
                .withMethod(HttpMethod.PUT)
                .withExpiration(expiration)

        return s3Config.amazonS3Client().generatePresignedUrl(generatePresignedUrlRequest).toString()
    }


    fun downloadPreSignedUrl(folder: String, fileName: String): Map<String, Serializable>? {
        val encodedFileName = "${fileName}"
//      s3에 실제 저장될 경로
        val objectKey = "${folder}/${encodedFileName}"

        val expiration = Date()
        var expTimeMillis: Long = expiration.time
        expTimeMillis += (3 * 60 * 1000).toLong() // 3분
        expiration.time = expTimeMillis // url 만료 시간 설정

        val generatePresignedUrlRequest: GeneratePresignedUrlRequest =
            GeneratePresignedUrlRequest(bucket, objectKey)
                .withMethod(HttpMethod.GET)
                .withExpiration(expiration)

        return mapOf(
            "preSignedUrl" to s3Config.amazonS3Client().generatePresignedUrl(generatePresignedUrlRequest),
            "encodedFileName" to encodedFileName
        )
    }


}
```

S3 Service 파일에 위의 코드를 추가한다.

presignedUrl을 생성하고, 다운로드하는 코드이다.

생성된 url로 프런트에서 이미지를 업로드하면 된다.

```kotlin
@RestController
class S3Controller(private val s3Service: S3Service) {

    @PostMapping("/s3/preSignedUrl")
    private fun getPreSignedUrl(
        @RequestParam folder: String,
        @RequestParam fileName: String
    ): String {
        return s3Service.getPreSignedUrl(folder, fileName)
    }


    @GetMapping("/s3/download")
    private fun downloadPreSignedUrl(
        @RequestParam folder: String,
        @RequestParam fileName: String
    ): Map<String, Serializable>? {
        return s3Service.downloadPreSignedUrl(folder, fileName)
    }


}
```

S3 Controller

<br>
## **3. 테스트**
---
이제 다 완성됐으니, post man으로 테스트해 보자.

![](https://blog.kakaocdn.net/dn/bP6K10/btsqtaCwktR/cRAU0KkRgBcwR2Rvk1ESUk/img.png)

presignedUrl이 잘 생성되었다.

![](https://blog.kakaocdn.net/dn/bZeEDW/btsqtyQGwQe/bDNuotE3ImrqIC0pwWbQm0/img.png)

생성된 url로 이미지를 업로드할 때는 put으로 해줘야 한다.

binary로 이미지를 업로드해 봤다.

![](https://blog.kakaocdn.net/dn/AE0ZN/btsqrFJB5jt/VogIAdjTFjeeU5mgNHb7K0/img.png)

업로드 완료!!!

<br>
## **4. 마치며**

presignedUrl은 delete를 지원하지 않기 때문에 이미지를 지우려면 따로 설정해줘야 한다.

이걸 몰라서 한참 헤맸다..

[https://docs.aws.amazon.com/AmazonS3/latest/userguide/uploading-downloading-objects.html](https://docs.aws.amazon.com/AmazonS3/latest/userguide/uploading-downloading-objects.html)

Pre-signed URL is supported for:

-   GET
-   PUT

It is not supported for:

-   LIST
-   COPY
-   DELETE
