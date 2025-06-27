# Table of contents
1. Build an interface on HTML -> To send data
2. Download Modbus-Serial lib -> Communication between JS and Modbus
3. Write a Modbus TCP Client  -> Write to Modbus
4. Write a Modbus TCP Slave   -> Modbus poll read data

# Mind map
✅ Nhập số trong HTML
✅ Gửi số đó về Node.js
✅ Node.js đóng vai Modbus Slave
✅ Modbus Poll (là Master) sẽ đọc được số đó qua TCP

# Build an interface on HTML
```
<!DOCTYPE html>
<html lang="en">
<head><meta charset="UTF-8"><title>Send to Modbus</title></head>
<body>
  <h2>Nhập số cần gửi:</h2>
  <input type="number" id="val" value="123">
  <button onclick="send()">Gửi</button>

  <script>
    async function send() {
      const v = +document.getElementById('val').value || 0;
      await fetch('/api/write', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({ value: v })
      });
      alert('Đã gửi ' + v);
    }
  </script>
</body>
</html>
```

# Download Modbus-Serial library
- GitHub: https://github.com/epsilonrt/modbus-serial

# Write a Modbus TCP Client
```
// file server.js
const express = require('express');
const ModbusRTU = require('modbus-serial');
const app = express().use(express.json());

const TARGET_IP = '192.168.1.50';  // địa chỉ thiết bị Modbus TCP
const UNIT_ID   = 1;               // slave ID
const REG_ADDR  = 0;               // Holding Register 40001

const mb = new ModbusRTU();
(async () => {
  await mb.connectTCP(TARGET_IP, { port: 502 });
  mb.setID(UNIT_ID);
})();

app.post('/api/write', async (req, res) => {
  try {
    const value = +req.body.value || 0;
    await mb.writeRegister(REG_ADDR, value); // FC=06
    res.json({ ok: true });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});

app.listen(3000, _=> console.log('Web → Modbus TCP client chạy cổng 3000'));
```
# Write a Modbus TCP Slave
```
// file slave.js
const express = require('express');
const { serverTCP } = require('modbus-serial');
const app = express().use(express.json());

const REG_ADDR = 0;
const holding  = Buffer.alloc(100 * 2); // 100 thanh ghi (2 byte mỗi thanh)

serverTCP((client) => {
  console.log('Modbus master connected');
}, {
  holding, 
  host: '0.0.0.0',
  port: 502
});

app.post('/api/write', (req, res) => {
  const v = +req.body.value || 0;
  holding.writeUInt16BE(v, REG_ADDR * 2);
  res.json({ ok: true });
});

app.listen(3000, _=> console.log('Web → Modbus TCP slave chạy cổng 3000'));
```
