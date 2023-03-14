接口
===============

初始化请求参数
------------------
::

    // 构建请求
    HttpRequest httpRequest = HttpUtil.createPost(uri + apiName);
    // 构建请求头
    Map<String ,String> headers = new HashMap<>();
    headers.put("request-id", requestId);
    headers.put("app-id", appId);
    headers.put("nonce",nonce);
    headers.put("signature",signatureData);
    httpRequest.addHeaders(headers);

hash存证（sha256） - /attestation/hash
------------------------------------
用户进行hash存证。

请求参数
^^^^^^^^^^^^^^^
=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
label               文件标签                                    非必填
hashItemList        HashItem对象列表                            必填
HashItem.fileName   文件名                                     必填
HashItem.hash       文件hash                                   必填
=================  ======================================= ================

返回的data
^^^^^^^^^^^^^^

调用hash存证接口成功后会返回存证id列表

===================  ================================
字段名 				    描述
===================  ================================
list                    bean对象列表
bean.hash               文件hash
bean.attestationId      存证id
===================  ================================

以java为例::

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

返回结果示例:
a.接口调用成功，则返回JSON数据示例为：::

    {
      "flag": true,
      "data": {
        "result": [
          {
            "hash": "98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1653",
            "ano": 840175805404684288
          }
        ]
      },
      "statusCode": "000000",
      "errorMessage": "操作成功"
    }

b.接口调用失败，则返回JSON数据示例为：::

    {
      "flag": false,
      "statusCode": "500087",
      "errorMessage": "hash错误"
    }

存证列表 - /attestation/list
----------------------

获取存证列表

请求参数
^^^^^^^^^^^^^^^
=================  ============================================ ============
参数名 				描述                                          是否可选
=================  ============================================ ============
attestationType     存证类型 8.hash存证   默认为8                     非必选
startTime           开始时间                                         非必选
endTime             结束时间                                         非必选
pageNum             当前页码                                         非必选
pageSize            每页显示数量 最大20                                非必选
fileName            文件名称                                         非必选
fileLabel           文件标签                                         非必选
fileHash            文件hash                                         非必选
channel             存证渠道：1.自助存证  2.api存证                     非必选
=================  ============================================ ============


返回的data
^^^^^^^^^^^^^^

调用存证获取列表接口成功后会返回存证列表

=====================  ===========================================================
字段名 				    描述
=====================  ===========================================================
pageNum                 当前页
pageSize                每页显示数量
pages                   总页数
size                    总数
total                   当前返回数量
list                    存证数据对象info
info.ano                存证编号
info.fileHash           文件hash
info.fileLabel          文件标签
info.fileName           文件名
info.createTime         创建时间
info.attestationType    存证类型 8:hash存证
info.state              1.上链中,2.上链失败,3.上链成功
=====================  ===========================================================


以java为例::

    // API path
    String apiName = "/attestation/list";
    HttpRequest httpRequest = createRequestPost(apiName);
    // 构建请求参数
    Map<String ,Object> body = new HashMap<>();
    httpRequest.body(JSONUtil.toJsonStr(body));
    String result;
    try (HttpResponse httpResponse = httpRequest.execute()) {
        result = httpResponse.body();
    }
    JSON json = JSONUtil.parse(resulY

返回结果示例:
a.接口调用成功，则返回JSON数据示例为：::

    {
      "flag": true,
      "data": {
        "total": 3,
        "list": [
          {
            "ano": "842425553641676801",
            "fileName": "test1",
            "fileLabel": "标签",
            "createTime": "2023-03-14 15:38:23",
            "fileHash": "98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1653",
            "attestationType": 8,
            "status": 3
          },
          {
            "ano": "842357643523006464",
            "fileName": "test1",
            "fileLabel": "标签",
            "createTime": "2023-03-14 11:08:32",
            "fileHash": "98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1653",
            "attestationType": 8,
            "status": 3
          },
          {
            "ano": "840600367778897920",
            "fileName": "test1",
            "fileLabel": "标签",
            "createTime": "2023-03-09 14:45:11",
            "fileHash": "98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1653",
            "attestationType": 8,
            "status": 3
          }
        ],
        "pageNum": 1,
        "pageSize": 20,
        "size": 3,
        "startRow": 1,
        "endRow": 3,
        "pages": 1,
        "prePage": 0,
        "nextPage": 0,
        "isFirstPage": true,
        "isLastPage": true,
        "hasPreviousPage": false,
        "hasNextPage": false,
        "navigatePages": 8,
        "navigatepageNums": [
          1
        ],
        "navigateFirstPage": 1,
        "navigateLastPage": 1
      },
      "statusCode": "000000",
      "errorMessage": "操作成功"
    }

b.接口调用失败，则返回JSON数据示例为：::

    {
      "flag": false,
      "statusCode": "500000",
      "errorMessage": "系统错误"
    }

存证详情 - /attestation/info
----------------------

查询存证详情。

请求参数
^^^^^^^^^^^^^^^

=================  ======================================= ================
参数名 				描述                                    是否可选
=================  ======================================= ================
ano                  存证编号                                   必选
type                 存证类型 8:hash存证 默认为8                  非必选
=================  ======================================= ================

返回的data
^^^^^^^^^^^^^^

调用存证详情成功后会返回详情数据

=======================  ================================
字段名 				        描述
=======================  ================================
ano                         存证编号
fileHash                    文件hash
label                       文件标签
fileName                    文件名
createTime                  创建时间
attestationType             存证类型 8:hash存证
status                      1.上链中,2.上链失败,3.上链成功
pdfUrl                      存证证书下载地址
blockchainHash              链hash
=======================  ================================


以java为例::

	String apiName = "/attestation/info";
    HttpRequest httpRequest = createRequestPost(apiName);
    // 构建请求参数
    Map<String ,Object> body = new HashMap<>();
    body.put("id","840175805404684288");
    httpRequest.body(JSONUtil.toJsonStr(body));
    String result;
    try (HttpResponse httpResponse = httpRequest.execute()) {
        result = httpResponse.body();
    }

返回结果示例:
a.接口调用成功，则返回JSON数据示例为：::

    {
      "flag": true,
      "data": {
        "ano": "842357643523006464",
        "status": 3,
        "category": 8,
        "createTime": "2023-03-14 11:08:32",
        "fileHash": "98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1653",
        "fileName": "test1",
        "label": "标签",
        "pdfUrl": "https://hbq.obs.cn-east-3.myhuaweicloud.com/staging/pdf/842357643523006464.pdf"
      },
      "statusCode": "000000",
      "errorMessage": "操作成功"
    }

b.接口调用失败，则返回JSON数据示例为：::

    {
      "flag": false,
      "statusCode": "520001",
      "errorMessage": "未查询到数据"
    }

