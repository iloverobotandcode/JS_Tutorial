#  Giải pháp tổng quát:
1. HTML nhập dữ liệu (client) 
2. Node.js server trung gian
3. Modbus Slave (giả lập) chạy trên Node.js
4. Modbus Poll kết nối đến Node.js

# Cách làm
## 1. Tạo project
- Bước 1.1: Mở VSCode hoặc phần mềm tương tự
- Bước 1.2: Tạo 1 file index.js ==> Mục đích: Mô phỏng modbus Slave (server)
- Bước 1.3: Tạo 1 file index.html ==> Mục đích: Nhập giá trị từ trang web [local host] (client)
- Bước 1.4: Chạy những đoạn chương trình sau trong VSCode => Terminal ==> Mục đích: Tạo các file JSON
  - Lưu ý: Các là phải tạo trong cùng thư mục với project hiện tại
  - Cài đặt thư viện JSON `npm install express body-parser modbus-serial`
## 2. Viết chương trình nhập số liệu HTML
```
<!-- index.html -->
<h2>Gửi giá trị Modbus từ Web</h2>
<input type="number" id="modbusValue" placeholder="Nhập giá trị...">
<button onclick="sendToModbus()">Gửi</button>

<script>
  function sendToModbus() {
    const value = document.getElementById("modbusValue").value;

    fetch("http://localhost:3000/write", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ value: parseInt(value) })
    })
    .then(res => res.text())
    .then(msg => alert(msg))
    .catch(err => alert("Lỗi: " + err));
  }
</script>
```
## 3. Viết chương trình Node.js
```
// server.js
const express = require("express");
const bodyParser = require("body-parser");
const net = require("net");
const ModbusRTU = require("modbus-serial");
const app = express();
const PORT = 3000;
const MODBUS_PORT = 5020; // dùng cổng khác 502 để tránh lỗi quyền admin

let holdingRegister = [0]; // thanh ghi giữ giá trị

app.use(bodyParser.json());
app.use(express.static(__dirname)); // phục vụ index.html

app.post("/write", (req, res) => {
  const value = req.body.value;
  if (typeof value === "number") {
    holdingRegister[0] = value;
    console.log("Đã nhận giá trị:", value);
    res.send("✔ Đã ghi giá trị: " + value);
  } else {
    res.status(400).send("Giá trị không hợp lệ");
  }
});

// Tạo Modbus TCP Slave server
const modbusServer = new ModbusRTU.ServerTCP({
  getHoldingRegisters: function(addr, length, cb) {
    cb(null, holdingRegister.slice(addr, addr + length));
  }
}, {
  host: "127.0.0.1",
  port: MODBUS_PORT,
  debug: true,
  unitID: 1
});

modbusServer.on("socketError", err => {
  console.error("Lỗi socket:", err);
});

app.listen(PORT, () => {
  console.log(`🟢 Web server đang chạy tại http://localhost:${PORT}`);
  console.log(`🟢 Modbus TCP Server đang lắng nghe trên cổng ${MODBUS_PORT}`);
});
```
## 4. Cấu hình Modbus poll

## 5. Thử nghiệm

## 6. Các lỗi sai
