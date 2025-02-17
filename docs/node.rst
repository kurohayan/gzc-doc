Node JS 示例代码

const { SM3 } = require("gm-crypto");
const fetch = require("node-fetch");

const requestId = generateUUId();
const appId = "xxxx";
const securityKey = "xxxxx";

const nonce = Math.floor(Date.now() / 1000);

const data = `${requestId}${appId}${nonce}`;

const b1 = Buffer.from(securityKey + data, "utf-8");

// 加密
const signatureData = SM3.digest(b1, undefined, "hex");

console.log(`signatureData: ${signatureData}`);

console.log(`requestId: ${requestId}`);
console.log(`appId: ${appId}`);
console.log(`nonce: ${nonce}`);

// 存证列表
const ListAPI = "https://xxxxx/shimakaze/api/attestation/list";

main();

async function main() {
  const res = await fetch(ListAPI, {
    method: "POST",
    headers: {
      "request-id": requestId,
      "app-id": appId,
      signature: signatureData,
      nonce: nonce,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({}),
  });

  const data = await res.json();
  console.log(data);
}

function generateUUId() {
  const chars = "0123456789abcdefghijklmnopqrstuvwxyz";
  let uuid = "";
  for (let i = 0; i < 32; i++) {
    const randomIndex = Math.floor(Math.random() * chars.length);
    uuid += chars[randomIndex];
  }
  return uuid;
}
