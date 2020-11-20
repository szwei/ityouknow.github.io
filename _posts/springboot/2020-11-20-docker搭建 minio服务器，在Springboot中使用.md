---
layout: post
title: docker搭建 minio服务器，在Springboot中使用
category: springboot
tags: [springboot]
---


### 一、介绍

>MinIO 是一个基于Apache License v2.0开源协议的对象存储服务。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。
>
>MinIO是一个非常轻量的服务,可以很简单的和其他应用的结合，类似 NodeJS, Redis 或者 MySQL

### 二、开始安装

1. ````
   #搜索镜像
   docker search minio
   ````

2. ````
   #拉取最新的镜像
   docker pull minio/minio
   ````

3. ````
   #查看拉取的镜像
   docker iamges
   ````

4. ````
   #自定义账号、密码。永久存储方式后台启动容器
   
   docker run -p 9000:9000 --name minio
   -d
   -e "MINIO_ACCESS_KEY=admin"
   -e "MINIO_SECRET_KEY=admin123456"
   -v /mnt/data:/data
   -v /mnt/config:/root/.minio
   minio/minio server /data
   
   ````

5. ````
   #查看容器启动状况
   docker ps 
   
   #查看日志
   docker logs <容器ID>
   ````

6. 在外部访问 IP:9000 访问即可

### 三、可能问题

1. 启动报错

   ````
    Access key length should be at least 3, and secret key length at least 8 characters
   ````

   说明用户名和不能少于3个字符，密码不能少于 8个 字符

   

**接下来就是在Springboot 项目中的应用了**

### 四、Springboot 中使用

1. 首先我们得新建一个Springboot 项目，这里不再赘述，传送门-> [原来这就是Springboot](https://mp.weixin.qq.com/s?__biz=MzU2Mjg3Mzk5Mg==&mid=2247483670&idx=1&sn=7326b09c001a9af2337a461e6e6b56ab&chksm=fc6392f1cb141be78b42a9a74975a9cda2376c1c48d22e7b2d746d113686c7623c515a076f67&mpshare=1&scene=1&srcid=1120bPZQKT0mBUC6Hj7sDKvH&sharer_sharetime=1605863868122&sharer_shareid=67913ab3f97507d6fe70e63c560a2063&key=d7bf35ae83f1a6f8176b177997c5e058c1987266c7ec3fa1239859c8e5ac664b0657eec821d398ca17394e31fd60f26e977f9c90ddf493f5959866d7dd477eb6ba535f9da5d0283dc09fe96eb938558e97e7c3327f5720a23e702a3242a543c6a3e7aa622fd37ba549543649e757668216c84275b9280e086374ce67634b29a6&ascene=1&uin=MjMwMDEyNzU3OQ%3D%3D&devicetype=Windows+10+x64&version=62090529&lang=zh_CN&exportkey=AbW73pXACC4yiOB6kIIqOCk%3D&pass_ticket=EDqqt6NR9s8YiBD4TsbM1Qc4fOZfVdFgFOPAkM8aKIjTRRRDXM68yiAVAXOw4g0W&wx_header=0)

2. 然后我们需要引入 依赖

    

   ````
      <dependency>
          <groupId>io.minio</groupId>
          <artifactId>minio</artifactId>
          <version>7.0.0</version>
   </dependency>
   ````

   官网的最新版本是 8.0.1 ，引用了最新的版本之后，发现很多方法都更新了，官方文档也没有及时的更新，qwq，还是用之前的稳定版本吧

3. 新建 MinioProperties ，从配置文件里取参数

   ````
   /**
    * minio 配置信息
    *
    */
   @Data
   @Configuration
   @ConfigurationProperties(prefix = "minio")
   public class MinioProperties {
   
   	private String endpoint;
   
   	private int port;
   
   	private String accessKey;
   
   	private String secretKey;
   
   	private boolean secure;
   
   	private String bucketName;
   
   }
   ````

   application.yml 内容：

   ````
   server:
     port: 8089
   
   
   minio:
     endpoint: 192.168.1.130
     port: 9000
     accessKey: admin
     secretKey: admin123456
     secure: false
     bucketName: "test123"
   ````

4. MinioConfiguration 配置类， 这里配置一个 minioTemplate，给他设置上参数

   ````
   @AllArgsConstructor
   @EnableConfigurationProperties({MinioProperties.class})
   public class MinioConfiguration {
   
       private final MinioProperties properties;
   
       @Bean
       MinioTemplate minioTemplate() {
           return new MinioTemplate(this.properties);
       }
   }
   
   ````

5. MinioTemplate，文件操作类，对Minio 提供的基本方法进行封装

   这里我们实现了 InitializingBean 接口，实现  **afterPropertiesSet**()  方法 在Spring 加在的时候，会调用Minio 的

   MinioClient 对象，完成初始化。

   ````
   @Component
   public class MinioTemplate implements InitializingBean {
   
       private MinioProperties minioProperties;
   
       private MinioClient client;
   
       /**
        * 创建存储桶
        *
        * @param bucketName bucket名称
        */
       @SneakyThrows
       public void createBucket(String bucketName) {
           if (!client.bucketExists(bucketName)) {
               client.makeBucket(bucketName);
           }
       }
   
       /**
        * 列出所有存储桶名称
        *
        * @return
        */
       @SneakyThrows
       public List<String> listBucketNames() {
           List<Bucket> bucketList = getAllBuckets();
           List<String> bucketListName = new ArrayList<>();
           for (Bucket bucket : bucketList) {
               bucketListName.add(bucket.name());
           }
           return bucketListName;
       }
   
       /**
        * 列出存储桶中的所有对象
        *
        * @param bucketName 存储桶名称
        * @return
        */
       @SneakyThrows
       public Iterable<Result<Item>> listObjects(String bucketName) {
           if (client.bucketExists(bucketName)) {
               return client.listObjects(bucketName);
           }
           return null;
       }
   
       /**
        * 获取全部存储桶
        * <p>
        */
       @SneakyThrows
       public List<Bucket> getAllBuckets() {
           return client.listBuckets();
       }
   
       /**
        * 根据名称获取存储桶
        * @param bucketName bucket名称
        */
       @SneakyThrows
       public Optional<Bucket> getBucket(String bucketName) {
           return client.listBuckets().stream().filter(b -> b.name().equals(bucketName)).findFirst();
       }
   
       /**
        * 根据名称删除存储桶
        * @param bucketName bucket名称
        */
       @SneakyThrows
       public void removeBucket(String bucketName) {
           Iterable<Result<Item>> myObjects = listObjects(bucketName);
           for (Result<Item> result : myObjects) {
               Item item = result.get();
               // 有对象文件，则删除失败
               if (item.size() > 0) {
                   return;
               }
           }
           // 删除存储桶，注意，只有存储桶为空时才能删除成功。
           client.removeBucket(bucketName);
       }
   
   
       /**
        * 通过InputStream上传对象
        *
        * @param bucketName bucket名称
        * @param objectName 文件名称
        * @param stream     文件流
        */
       public void putObject(String bucketName, String objectName, InputStream stream) throws Exception {
           if (client.bucketExists(bucketName)) {
               client.putObject(bucketName, objectName, stream, new PutObjectOptions(stream.available(), -1));
               ObjectStat statObject = getObjectInfo(bucketName, objectName);
               if (statObject != null && statObject.length() > 0) {
               }
           }
       }
   
       /**
        * 上传文件
        *
        * @param bucketName  bucket名称
        * @param objectName  文件名称
        * @param filename    文件名
        */
       public void putObject(String bucketName, String objectName, String filename) throws Exception {
           client.putObject(bucketName,objectName,filename,null);
       }
   
       /**
        * 文件上传
        *
        * @param bucketName
        * @param multipartFile
        */
       @SneakyThrows
       public void putObject(String bucketName, MultipartFile multipartFile, String filename) {
           PutObjectOptions putObjectOptions = new PutObjectOptions(multipartFile.getSize(), PutObjectOptions.MIN_MULTIPART_SIZE);
           putObjectOptions.setContentType(multipartFile.getContentType());
           client.putObject(bucketName, filename, multipartFile.getInputStream(), putObjectOptions);
       }
   
   
       /**
        * 获取文件外链
        *
        * @param bucketName bucket名称
        * @param objectName 文件名称
        * @param expires    过期时间 <=7
        * @return url
        */
       @SneakyThrows
       public String getObjectURL(String bucketName, String objectName, Integer expires) {
           return client.presignedGetObject(bucketName, objectName, expires);
       }
   
       /**
        *  以流的形式获取一个文件对象
        *
        * @param bucketName bucket名称
        * @param objectName 文件名称
        * @return 二进制流
        */
       @SneakyThrows
       public InputStream getObject(String bucketName, String objectName) {
           return client.getObject(bucketName, objectName);
       }
   
   
       /**
        * 获取文件信息
        *
        * @param bucketName bucket名称
        * @param objectName 文件名称
        */
       public ObjectStat getObjectInfo(String bucketName, String objectName) throws Exception {
           return client.statObject(bucketName, objectName);
       }
   
       /**
        * 删除文件
        *
        * @param bucketName bucket名称
        * @param objectName 文件名称
        */
       public void removeObject(String bucketName, String objectName) throws Exception {
           client.removeObject(bucketName, objectName);
       }
   
   
       @Override
       public void afterPropertiesSet() throws Exception {
           this.client = new MinioClient(minioProperties.getEndpoint(),
                   minioProperties.getPort(),
                   minioProperties.getAccessKey(),
                   minioProperties.getSecretKey(),
                   minioProperties.isSecure());
       }
   
       public MinioTemplate(MinioProperties minioProperties){
           this.minioProperties = minioProperties;
       }
   }
   ````

6. 测试 Controller

   ````
   @AllArgsConstructor
   @RestController
   @RequestMapping("/oss")
   public class TestController {
   
       private final MinioTemplate minioTemplate;
       private final MinioProperties minioProperties;
   
       @SneakyThrows
       @PostMapping("/upload")
       public Map<String, String> upload(@RequestParam("file") MultipartFile file) {
           String filename = file.getOriginalFilename();
           String bucketName = minioProperties.getBucketName();
           String newFileName = IdUtil.getSnowflake(1,1).nextId() + StrUtil.DOT + FileUtil.extName(file.getOriginalFilename());
   
           minioTemplate.createBucket(bucketName);
   
           minioTemplate.putObject(bucketName, newFileName, file.getInputStream());
   
           Map<String, String> map = new HashMap<>(4);
           map.put("bucketName", bucketName);
           map.put("newFileName", newFileName);
           map.put("filename", filename);
           map.put("url", "http://localhost:8089/oss/"+newFileName);
           return map;
       }
   
   
       /**
        * 获取文件 直接重定向到对应服务
        * 用户服务要能访问到文件服务器
        *
        * @param fileName 文件空间/名称
        * @param response
        * @return
        */
       @SneakyThrows
       @GetMapping("/real/{fileName:.+}")
       public void realfileUrl(@PathVariable String fileName, HttpServletResponse response) {
           String bucketName = minioProperties.getBucketName();
           String url = minioTemplate.getObjectURL(bucketName, fileName, 7);
           response.sendRedirect(url);
       }
   
       /**
        * 直接获取文件下载流
        *
        * @param fileName
        * @param response
        */
       @SneakyThrows
       @GetMapping("/{fileName:.+}")
       public void file(@PathVariable String fileName, HttpServletResponse response) {
           String bucketName = minioProperties.getBucketName();
           InputStream inputStream = minioTemplate.getObject(bucketName, fileName);
           IoUtil.copy(inputStream, response.getOutputStream());
       }
   
   }
   ````

7. 测试结果，使用 postman 上传文件

   ````
   {
       "bucketName": "test123",
       "newFileName": "1329700843150249984.jpg",
       "filename": "5b4412b8N28240fd4.jpg",
       "url": "http://localhost:8089/oss/1329700843150249984.jpg"
   }
   ````

  