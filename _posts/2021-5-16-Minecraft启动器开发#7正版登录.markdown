---
layout: post
title: "Minecraft启动器开发#7正版登录"
data: 2021-5-16 21:42
categories: java
---

```java
public static String doPost(URL url, String post) throws IOException {
    byte[] bytes = post.getBytes(StandardCharsets.UTF_8);
    HttpURLConnection con = (HttpURLConnection) url.openConnection();
    con.setRequestMethod("POST");
    con.setDoOutput(true);
    con.setRequestProperty("Content-Type", "application/json; charset=utf-8");
    con.setRequestProperty("Content-Length", String.valueOf(bytes.length));
    IOUtils.write(bytes, con.getOutputStream());
    return IOUtils.toString(con.getInputStream(), StandardCharsets.UTF_8);
}
```

发送post请求

请求地址`https://authserver.mojang.com/authenticate`

类型为`application/json`

```json
{
    "agent": {                              // defaults to Minecraft
        "name": "Minecraft",                // For Mojang's other game Scrolls, "Scrolls" should be used
        "version": 1                        // This number might be increased
                                            // by the vanilla client in the future
    },
    "username": "mojang account name",      // Can be an email address or player name for
                                            // unmigrated accounts
    "password": "mojang account password",
    "clientToken": "client identifier",     // optional
    "requestUser": true                     // optional; default: false; true adds the user object to the response
}
```

参数

```json
{
    "user": {
        "username": "user@email.example", // will be account username for legacy accounts
        "properties": [
            {
                "name": "preferredLanguage",
                "value": "en-us"
            },
            {
                "name": "registrationCountry",
                "value": "country" // 2L country (e.g. US)
            }
        ],
        "id": "hexadecimal string" // Different from the UUID, not sure what it is?
    },
    "clientToken": "client identifier",
    "accessToken": "random access token", // hexadecimal or JSON-Web-Token (unconfirmed) [The normal accessToken can be found in the payload of the JWT (second by '.' separated part as Base64 encoded JSON object), in key "yggt"]
    "availableProfiles": [
        {
            "name": "player username",
            "id": "hexadecimal string" // UUID of the account
        }
    ],
    "selectedProfile": {
        "name": "player username",
        "id": "hexadecimal string" // UUID of the account
    }
}
```

返回示例

```java
var param = """
        {
            "agent": {
                "name": "Minecraft",
                "version": 1
            },
            "username": "${username}",
            "password": "${password}",
            "clientToken": "${uuid}",
            "requestUser": true
        }
        """;
```

将参数替换为相对应的值

```java
param = param.replace("${uuid}", UUID.randomUUID().toString());
```

随机生成UUID

```java
var ret = gson.fromJson(Util.doPost(new URL("https://authserver.mojang.com/authenticate"), param), JsonObject.class);
var selectedProfile = ret.get("selectedProfile").getAsJsonObject();
username = selectedProfile.get("name").getAsString();
uuid = selectedProfile.get("id").getAsString();
accessToken = ret.get("accessToken").getAsString();
```

获取返回赋值