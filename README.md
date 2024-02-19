# bennkyou-you
ESP32をサーバーに通信するプログラムのデモ

基本的にプログラム内のコメントアウトを参照

## Rust（手元のPC側）
```rs
use std::net::UdpSocket;
use std::time::Duration;

fn main() {
    println!("Start UDP");
    // 指定したアドレスとポートでスタート
    let udp = UdpSocket::bind("192.168.4.2:8080").unwrap();

    // 必要な変数の用意
    let mut count = 0;
    // バッファを用意
    let mut buf = [0_u8;256];
    println!("Start Loop");
    loop {
        // 送る文字列を作成
        let msg = format!("Hello,{}", count);

        // 「文字列をバイトに変換して送りたい送り先に送信」が成功したかを確認するmatch
        match udp.send_to(msg.as_bytes(), "192.168.4.1:10000")
        {
            // 成功したら送信に成功したバイト数が出る
            Ok(size)=>{
                println!("Send {}, size:{}", msg, size);
            }
            // 失敗したらエラー文が返される
            Err(e)=>{
                println!("Failed to send:{}", e);
            }
        }

        // 「受信した内容をバッファに書き込む」が成功したか確認するmatch
        match udp.recv_from(&mut buf)
        {
            // 成功したら受信したサイズと受信したパケットの送信元が出る
            Ok((size, dest))=>{
                // 受信したバイト列の最後に終わりを示すヌル文字を入れる
                buf[size] = b'\0';

                // バイト列をString(文字列)に変換
                let msg = String::from_utf8_lossy(&buf).to_string();
                println!("Recieve:{}, from {}", msg, dest.to_string());
            }
            // 失敗したらエラー文が返される
            Err(e)=>{
                println!("Failed to recieve:{}", e);
            }
        }

        // カウントを１ずつ増加
        count += 1;

        // プログラムを1秒停止
        std::thread::sleep(Duration::from_millis(1000));
    }
}
```

## ESP32側（Arduino）

```cpp
#include <WiFi.h>
#include <WiFiUdp.h>

// コード全体で使う変数の宣言
// WiFiの名前の宣言
const char* ssid = "TestServer";
// WiFiのパスワードを設定
const char* pass = "test10000";
// ポートを指定
const uint16_t port = 10000;
// サーバーのアドレス等を定数(constant)として設定する
const IPAddress ip(192, 168, 4, 1);
const IPAddress gateway(192, 168, 4, 1);
const IPAddress subnet(255, 255, 255, 0);

// 通信相手のアドレス、ポート
const IPAddress remote_ip(192, 168, 4, 2);
const uint16_t remote_port = 8080;

// UDP通信用の変数を初期化
WiFiUDP udp;

// 受信用のバッファを用意する
char buf[256];

void setup() {
  // デバック用のログを表示。
  Serial.begin(115200);
  Serial.printf("Start Connecting");

  // WiFiの初期化
  WiFi.softAP(ssid, pass);
  WiFi.softAPConfig(ip, gateway, subnet);

  Serial.println("\nWiFi is Connected");

  // UDP通信スタート
  udp.begin(port);
  Serial.println("Start UDP");
}

void loop() {
  // 受信したパケットのサイズをsizeに入れる
  int size = udp.parsePacket();

  if (size) {
    // バッファを初期化（空にする）
    for (int i = 0 ; i < 256 ; i++ ) buf[i] = 0;
    // バッファに受信した内容を書き込み
    udp.read(buf, 256);
    // 受信した内容を表示
    Serial.println(buf);

    // 指定したアドレス、ポートに送信
    udp.beginPacket(remote_ip, remote_port);
    udp.printf("Hello, ESP32");
    udp.endPacket();
  }
}
```

## 成功映像

設定からESP32が出してるWi-Fi、このコードだと「TestServer」に接続する

https://github.com/motii8128/bennkyou-you/assets/108280115/60422048-7137-48c3-bf58-25ccbb914692

