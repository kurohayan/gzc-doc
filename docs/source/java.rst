Java
=================

如果使用maven，可以加入如下依赖::

	<dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.7.22</version>
    </dependency>
    <dependency>
        <groupId>org.bouncycastle</groupId>
        <artifactId>bcprov-jdk15to18</artifactId>
        <version>1.71</version>
    </dependency>
    <dependency>
        <groupId>com.huaweicloud</groupId>
        <artifactId>esdk-obs-java-bundle</artifactId>
        <version>[3.21.11,)</version>
    </dependency>


示例程序
------------------

java::

    import cn.hutool.core.util.IdUtil;
    import cn.hutool.core.util.RandomUtil;
    import cn.hutool.crypto.SecureUtil;
    import cn.hutool.crypto.digest.SM3;
    import cn.hutool.http.HttpRequest;
    import cn.hutool.http.HttpResponse;
    import cn.hutool.http.HttpUtil;
    import cn.hutool.json.JSON;
    import cn.hutool.json.JSONObject;
    import cn.hutool.json.JSONUtil;
    import com.obs.services.ObsClient;
    import com.obs.services.ObsConfiguration;
    import com.obs.services.model.PutObjectRequest;
    import com.obs.services.model.PutObjectResult;
    import org.junit.Test;

    import java.io.File;
    import java.nio.file.Files;
    import java.nio.file.Paths;
    import java.time.LocalDate;
    import java.util.*;

    public class ShimakazeApiRequestTest {

        private final String uri = "http://127.0.0.1:10000/shimakaze/api";

        /**
         *
         * @throws Exception
         */
        @Test
        public void detail() throws Exception {
            String apiName = "/attestation/info";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            Map<String ,Object> body = new HashMap<>();
            body.put("ano","842711150876827648");
            httpRequest.body(JSONUtil.toJsonStr(body));
            String result;
            try (HttpResponse httpResponse = httpRequest.execute()) {
                result = httpResponse.body();
            }
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void list() throws Exception {
            // API path
            String apiName = "/attestation/list";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            Map<String ,Object> body = new HashMap<>();
            body.put("channel","2");
            httpRequest.body(JSONUtil.toJsonStr(body));
            String result;
            try (HttpResponse httpResponse = httpRequest.execute()) {
                result = httpResponse.body();
            }
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void hash() throws Exception {
            // API path
            String apiName = "/attestation/hash";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            List<EvidenceHashParam.HashItem> list = new ArrayList<>();
            EvidenceHashParam.HashItem hashInfo1 = new EvidenceHashParam.HashItem();
            hashInfo1.setFileName("test1");
            hashInfo1.setHash("98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1654");
            list.add(hashInfo1);
            EvidenceHashParam evidenceHashParam = new EvidenceHashParam();
            evidenceHashParam.setLabel("标签");
            evidenceHashParam.setHashItemList(list);
            httpRequest.body(JSONUtil.toJsonStr(evidenceHashParam));
            String result;
            try (HttpResponse httpResponse = httpRequest.execute()) {
                result = httpResponse.body();
            }
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void file() throws Exception {
            // API path
            String apiName = "/attestation/file";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            httpRequest.form("file",new File("/tmp/123.jpg"));
            httpRequest.form("label","标签");
            String result;
            try (HttpResponse httpResponse = httpRequest.execute()) {
                result = httpResponse.body();
            }
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void web() throws Exception {
            // API path
            String apiName = "/attestation/web";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            ApiWebAttestationParam apiWebAttestationParam = new ApiWebAttestationParam();
            apiWebAttestationParam.setLabel("标签");
            apiWebAttestationParam.setUrl("https://www.baidu.com");
            apiWebAttestationParam.setName("取证-百度");
            httpRequest.body(JSONUtil.toJsonStr(apiWebAttestationParam));
            String result;
            try (HttpResponse httpResponse = httpRequest.execute()) {
                result = httpResponse.body();
            }
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void download() throws Exception {
            // API path
            String ano = "840175805404684288";
            String apiName = "/attestation/cert?ano=" + ano;
            HttpRequest httpRequest = createRequestGet(apiName);
            try (HttpResponse httpResponse = httpRequest.execute()) {
                if (httpResponse.getStatus() != 200) {
                    System.out.println("未查询到可用资源:" + httpResponse.body());
                    return;
                }
                String body = httpResponse.body();
                JSONObject jsonObject = JSONUtil.parseObj(body);
                String statusCode = jsonObject.getStr("statusCode");
                if ("000000".equals(statusCode)) {
                    JSONObject data = jsonObject.getJSONObject("data");
                    String pdfUrl = data.getStr("pdfUrl");
                    HttpUtil.download(pdfUrl, Files.newOutputStream(Paths.get("/tmp/" + ano + ".pdf")), true);
                } else {
                    System.out.println("未查询到可用资源:" + httpResponse.body());
                }
            }

        }

        @Test
        public void testSubmitEnforcerAttestation() throws Exception {
            String apiName = "/attestation/enforcer";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            File file = new File("/tmp/123.mp4");
            String ossKey = uploadOss(file);
            SubmitEnforcerRecordParam param = new SubmitEnforcerRecordParam();
            param.setName("test");
            param.setDeviceId("E123456");
            param.setAddress("地址");
            param.setLabel("标签");
            param.setEvidenceType(2);
            param.setStartTime("2023-04-04 13:10:12");
            param.setEndTime("2023-04-04 13:30:12");
            param.setSaveTime("2023-04-04 14:10:10");
            param.setFileHash(SecureUtil.sha256(file));
            param.setFileName(file.getName());
            param.setFileOssKey(ossKey);
            param.setFileSize(file.length());
            httpRequest.body(JSONUtil.toJsonStr(param));
            String result;
            try (HttpResponse httpResponse = httpRequest.execute()) {
                result = httpResponse.body();
            }
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        private HttpRequest createRequestPost(String apiName) throws Exception {
            // 构建请求
            HttpRequest httpRequest = HttpUtil.createPost(uri + apiName);
            setHttpRequestHeaders(httpRequest);
            return httpRequest;
        }
        private HttpRequest createRequestGet(String apiName) throws Exception {
            // 构建请求
            HttpRequest httpRequest = HttpUtil.createGet(uri + apiName);
            setHttpRequestHeaders(httpRequest);
            return httpRequest;
        }

        private HttpRequest setHttpRequestHeaders(HttpRequest httpRequest) throws Exception {
            // securityKey
            String securityKey = "689d7ff1ebf746389f65c32112c27c76";
            // 请求头
            String requestId = IdUtil.simpleUUID();
            String appId = "d29f2fd7a8dc42b4";
            String nonce = String.valueOf(System.currentTimeMillis() / 1000);

            //待签名数据 = requestId+accessKey+nonce
            String content = requestId + appId + nonce;
            SM3 sm3 = new SM3(securityKey.getBytes());
            String signatureData = sm3.digestHex(content);
            // 构建请求头
            Map<String ,String> headers = new HashMap<>();
            headers.put("request-id", requestId);
            headers.put("app-id", appId);
            headers.put("nonce",nonce);
            headers.put("signature",signatureData);
            httpRequest.addHeaders(headers);
            return httpRequest;
        }

        private String uploadOss(File file) throws Exception {
            String suffix = suffix(file.getName());
            String ossKey = "enforcer/" + LocalDate.now() + "/" + IdUtil.simpleUUID() + suffix;
            ObsClient obsClient = getObsClient();
            PutObjectRequest request = new PutObjectRequest();
            request.setBucketName("test");
            request.setObjectKey(ossKey);
            request.setFile(file);
            PutObjectResult result = obsClient.putObject(request);
            if (result.getStatusCode() == 200) {
                return ossKey;
            }
            throw new Exception("上传失败");
        }

        private String suffix(String fileName) {
            int num = fileName.lastIndexOf(".");
            if (num != -1) {
                return fileName.substring(num);
            } else {
                return null;
            }
        }

        private ObsClient getObsClient() {
            String ak = "ak";
            String sk = "sk";
            String endPoint = "https://obs.cn-east-3.myhuaweicloud.com";
            return new ObsClient(ak, sk, endPoint);
        }

        public static void main(String[] args) {
            // securityKey
            String securityKey = "689d7ff1ebf746389f65c32112c27c76";

            // 请求头
            String requestId = IdUtil.simpleUUID();
            String appId = "d29f2fd7a8dc42b4";
            long nonce = System.currentTimeMillis() / 1000;
            // API path
            //待签名数据 = requestId+appId+nonce
            String data = requestId + appId + nonce;
            // 开始签名
            SM3 sm3 = new SM3(securityKey.getBytes());
            String signatureData = sm3.digestHex(data);
            System.out.println(requestId);
            System.out.println(nonce);
            System.out.println(data);
            System.out.println(signatureData);
        }

         public class ApiWebAttestationParam {
            private String url;

            private String name;

            private String label;

            public String getUrl() {
                return url;
            }

            public void setUrl(String url) {
                this.url = url;
            }

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }

            public String getLabel() {
                return label;
            }

            public void setLabel(String label) {
                this.label = label;
            }
        }

        static class EvidenceHashParam {
            private String label;
            private List<HashItem> hashItemList;
            static class HashItem {
                private String hash;

                private String fileName;

                public String getHash() {
                    return hash;
                }

                public void setHash(String hash) {
                    this.hash = hash;
                }

                public String getFileName() {
                    return fileName;
                }

                public void setFileName(String fileName) {
                    this.fileName = fileName;
                }
            }

            public String getLabel() {
                return label;
            }

            public void setLabel(String label) {
                this.label = label;
            }

            public List<HashItem> getHashItemList() {
                return hashItemList;
            }

            public void setHashItemList(List<HashItem> hashItemList) {
                this.hashItemList = hashItemList;
            }
        }
        static class SubmitEnforcerRecordParam {
            /**
             * 执法记录仪编号
             */
            private String deviceId;
            /**
             * 文件hash
             */
            private String fileHash;
            /**
             * 取证名称
             */
            private String name;
            /**
             * 取证标签
             */
            private String label;
            /**
             * 取证类型:1.拍照取证，2.录像取证，3.录音取证
             */
            private Integer evidenceType;
            /**
             * 文件大小
             */
            private Long fileSize;
            /**
             * 文件名
             */
            private String fileName;
            /**
             * 文件上传到oss后的key
             */
            private String fileOssKey;
            /**
             * 取证地址
             */
            private String address;
            /**
             * 取证开始时间
             */
            private String startTime;
            /**
             * 取证结束时间
             */
            private String endTime;
            /**
             * 上传时间
             */
            private String saveTime;

            public String getDeviceId() {
                return deviceId;
            }

            public void setDeviceId(String deviceId) {
                this.deviceId = deviceId;
            }

            public String getFileHash() {
                return fileHash;
            }

            public void setFileHash(String fileHash) {
                this.fileHash = fileHash;
            }

            public String getName() {
                return name;
            }

            public void setName(String name) {
                this.name = name;
            }

            public String getLabel() {
                return label;
            }

            public void setLabel(String label) {
                this.label = label;
            }

            public Integer getEvidenceType() {
                return evidenceType;
            }

            public void setEvidenceType(Integer evidenceType) {
                this.evidenceType = evidenceType;
            }

            public Long getFileSize() {
                return fileSize;
            }

            public void setFileSize(Long fileSize) {
                this.fileSize = fileSize;
            }

            public String getFileName() {
                return fileName;
            }

            public void setFileName(String fileName) {
                this.fileName = fileName;
            }

            public String getFileOssKey() {
                return fileOssKey;
            }

            public void setFileOssKey(String fileOssKey) {
                this.fileOssKey = fileOssKey;
            }

            public String getAddress() {
                return address;
            }

            public void setAddress(String address) {
                this.address = address;
            }

            public String getStartTime() {
                return startTime;
            }

            public void setStartTime(String startTime) {
                this.startTime = startTime;
            }

            public String getEndTime() {
                return endTime;
            }

            public void setEndTime(String endTime) {
                this.endTime = endTime;
            }

            public String getSaveTime() {
                return saveTime;
            }

            public void setSaveTime(String saveTime) {
                this.saveTime = saveTime;
            }
        }

    }


