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


示例程序
------------------

java::

    import cn.hutool.core.util.IdUtil;
    import cn.hutool.crypto.digest.SM3;
    import cn.hutool.http.HttpRequest;
    import cn.hutool.http.HttpResponse;
    import cn.hutool.http.HttpUtil;
    import cn.hutool.json.JSON;
    import cn.hutool.json.JSONObject;
    import cn.hutool.json.JSONUtil;
    import org.junit.Test;

    import java.nio.file.Files;
    import java.nio.file.Paths;
    import java.util.*;

    public class ApiRequestTest {

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

        private final String uri = "http://127.0.0.1:8080/shimakaze/api";

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
            body.put("ano","840175805404684288");
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
    //        body.put("attestationId","");
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
            hashInfo1.setHash("98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1653");
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
            // securityKey（盐值）
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

    }


