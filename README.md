<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<title>TRON 靓地址生成器</title>

<script src="https://cdn.jsdelivr.net/npm/@noble/secp256k1@1.7.1/dist/index.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/js-sha3@0.8.0/dist/sha3.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bs58@5.0.0/dist/bs58.min.js"></script>

<style>
body { font-family: Arial; padding: 20px; }
input, button { padding: 8px; margin: 6px 0; width: 100%; }
pre { background: #f4f4f4; padding: 10px; }
</style>
</head>

<body>

<h2>TRON 靓地址生成器</h2>

<input id="prefix" placeholder="输入前缀（如 88 / Jack）">
<button onclick="generate()">开始生成</button>

<p id="status"></p>
<pre id="result"></pre>

<script>
function randomPrivateKey() {
  return crypto.getRandomValues(new Uint8Array(32));
}

function toHex(bytes) {
  return Array.from(bytes).map(b => b.toString(16).padStart(2, '0')).join('');
}

async function generate() {
  const prefix = document.getElementById("prefix").value;
  if (!prefix) return alert("请输入前缀");

  document.getElementById("status").innerText = "生成中，请稍候…";
  document.getElementById("result").innerText = "";

  while (true) {
    const priv = randomPrivateKey();
    const pub = secp256k1.getPublicKey(priv, false).slice(1);

    const hash = sha3_256(pub);
    const addressHex = "41" + hash.slice(-40);

    const addrBytes = Uint8Array.from(addressHex.match(/.{2}/g).map(b => parseInt(b, 16)));
    const check = sha3_256(addrBytes);
    const checksum = sha3_256(
      Uint8Array.from((addressHex + check.slice(0, 8)).match(/.{2}/g).map(b => parseInt(b, 16)))
    ).slice(0, 8);

    const base58 = bs58.encode(
      Uint8Array.from((addressHex + checksum).match(/.{2}/g).map(b => parseInt(b, 16)))
    );

    if (base58.startsWith("T" + prefix)) {
      document.getElementById("status").innerText = "生成成功 ✅";
      document.getElementById("result").innerText =
`地址: ${base58}
私钥: ${toHex(priv)}

⚠️ 请立即保存私钥`;
      break;
    }
  }
}
</script>

</body>
</html>
