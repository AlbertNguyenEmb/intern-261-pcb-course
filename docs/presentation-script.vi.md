# Kịch bản thuyết trình bộ kit STM32F407VET6

Cơ sở: schematic tại commit `cbc661b`. Thời lượng gợi ý: 12–15 phút.
Các đoạn trong ngoặc kép có thể đọc trực tiếp. Mỗi khối dùng bốn ý:
mục đích → linh kiện → đường tín hiệu → mức xác minh và phần còn thiếu.

## Mở đầu

“Đề tài của em là thiết kế bộ kit học tập sử dụng STM32F407VET6. Kit hướng
đến ba mức thực hành: điều khiển ngoại vi cơ bản; xây dựng driver, DMA và
FreeRTOS; sau đó trao đổi dữ liệu qua CAN và Ethernet với thiết bị khác.

Em trình bày từ khối nguồn và hệ thống tối thiểu, đến các khối thu thập,
xử lý, lưu trữ và truyền dữ liệu. Phần trình bày hôm nay mô tả thiết kế
schematic. Khi nói ‘đã kiểm tra kết nối’, em đang nói đến kiểm tra
schematic, netlist hoặc ERC; các kiểm tra này không thay thế kết quả đo
trên phần cứng.”

## 1. Nguồn — power.kicad_sch

1. **Cần giải quyết việc gì?**

   “Kit cần nhận nguồn từ adapter 12 V hoặc USB 5 V, lựa chọn nguồn cấp
   và tạo mức 3,3 V cho MCU cùng ngoại vi. Thiết kế cũng cần hạn chế tác
   động của cắm ngược cực và quá dòng.”

2. **Linh kiện nào thực hiện?**

   “F1 là cầu chì tự phục hồi; Q1 AO3401A và D1 zener 8,2 V thuộc mạch
   bảo vệ đầu vào. AP63205 chuyển 12 V xuống 5 V, TPS2116 chọn giữa hai
   nguồn 5 V, và AP63203 tạo nguồn 3,3 V. Cuộn cảm cùng tụ là thành phần
   của bộ biến đổi buck và mạch lọc nguồn.”

3. **Tín hiệu đi như thế nào?**

   “Nhánh adapter đi từ jack DC, qua bảo vệ đầu vào rồi qua buck 5 V.
   Nhánh USB đi qua bảo vệ USB rồi vào bộ chọn nguồn. Đầu ra chung
   `+5V_SYS` cấp cho buck 3,3 V và regulator của debugger. Nguồn `+3V3`
   cấp cho F407 và các ngoại vi; `+3V3_DBG` cấp riêng cho F103, nhưng
   các khối vẫn dùng chung GND. Hai tên nguồn không có nghĩa là cách ly.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Netlist đã xác nhận đầu ra sau L2 nối cùng mạng 3,3 V với nguồn MCU.
   Em chưa có kết quả đo điện áp, ripple, nhiệt độ, quá độ khi chuyển
   nguồn hoặc thử bảo vệ ngược cực. Cần chốt thông số linh kiện nguồn,
   tính tổng tải và kiểm tra các trạng thái adapter, USB và đồng thời
   cả hai trước khi kết luận khối nguồn hoạt động ổn định.”

## 2. MCU và hệ thống tối thiểu — sheet STM32F407VET6_Minimum_System

1. **Cần giải quyết việc gì?**

   “Khối này tạo điều kiện để MCU khởi động, có clock, nguồn và trạng
   thái boot xác định. Nó cũng là nơi phân bổ chân để các ngoại vi có
   thể cùng tồn tại trong một thiết kế.”

2. **Linh kiện nào thực hiện?**

   “U1 là STM32F407VET6, dùng nguồn 3,3 V. Y1 là thạch anh HSE 8 MHz.
   Các tụ decoupling hỗ trợ cấp dòng tức thời, hai tụ VCAP 2,2 µF phục
   vụ regulator lõi bên trong, và FB1 nằm trên đường cấp analog.
   Điện trở và công tắc BOOT0 dùng để chọn trạng thái khởi động.”

3. **Tín hiệu đi như thế nào?**

   “Clock từ thạch anh đi vào PH0/PH1. Nguồn số đi vào VDD; nguồn analog
   qua nhánh lọc đến VDDA và VREF+. Sau khi khởi động, MCU điều khiển
   từng khối thông qua GPIO, timer, I²C, SPI, SDIO, CAN và RMII. Tụ
   VCAP nối với các chân VCAP xuống GND; VCAP không phải đầu ra nguồn
   để cấp tải ngoài.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Đã đối chiếu nhiều đường ngoại vi với netlist và mapping chân
   LQFP100. HSE mới là thiết kế, chưa có phép đo khởi động oscillator.
   LSE/RTC, lựa chọn nguồn VBAT, BOOT1 và mạch reset target vẫn cần
   hoàn thiện hoặc rà soát. Hiện VBAT nối 3,3 V; nếu thêm pin dự phòng
   phải thiết kế lại đường cấp, không nối pin trực tiếp song song với
   nguồn này.”

## 3. Debugger onboard — onboard_debugger.kicad_sch

1. **Cần giải quyết việc gì?**

   “Debugger giúp máy tính nạp chương trình và theo dõi F407 qua SWD.
   Mục tiêu ban đầu còn có SWO và UART Virtual COM Port, nhưng khả năng
   thực tế phải khớp với firmware debugger được sử dụng.”

2. **Linh kiện nào thực hiện?**

   “Khối dùng STM32F103C8, regulator AMS1117-3.3, thạch anh 8 MHz,
   USBLC6 bảo vệ đường USB và các jumper ngắt kết nối sang target.
   J2 là đầu SWD để lập trình chính F103.”

3. **Tín hiệu đi như thế nào?**

   “Máy tính gửi dữ liệu USB vào F103. Firmware trên F103 chuyển lệnh
   thành tín hiệu điều khiển target: PB7 đi qua JP1 đến PA13/SWDIO của
   F407; PB8 qua JP2 đến PA14/SWCLK; PB9 qua JP3 đến NRST. Reset và
   đầu SWD để lập trình F103 là các đường riêng.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Đã kiểm tra các đường nối này trong netlist. Firmware CMSIS-DAP
   cụ thể vẫn chưa được xác nhận tương thích với pinout, nên chưa
   chứng minh được nạp và debug thực tế. Em gọi đây là debugger F103
   dự kiến dùng CMSIS-DAP, chưa gọi là ST-LINK. SWO và VCP chưa được
   hoàn thiện; cần firmware phù hợp và thử với công cụ trên máy tính.”

## 4. LED, nút nhấn và buzzer — basic_io.kicad_sch

1. **Cần giải quyết việc gì?**

   “Khối tạo bài tập nhập xuất số, đọc nút bằng polling hoặc ngắt,
   chống dội và phát tín hiệu bằng timer/PWM. Jumper cho phép ngắt
   tải onboard để sử dụng lại chân MCU.”

2. **Linh kiện nào thực hiện?**

   “Bốn LED có điện trở hạn dòng riêng, ba nút có điện trở kéo lên,
   và buzzer được điều khiển qua MOSFET Q3 BUK9832-55A. Mỗi nhánh
   thực hành có jumper liên quan để cấu hình kết nối.”

3. **Tín hiệu đi như thế nào?**

   “PE2 đến PE5 điều khiển bốn LED qua jumper và điện trở. Các nút
   đưa tín hiệu về PE0, PE1 và PE15; khi nhấn, tín hiệu được kéo xuống
   GND. PB0 xuất PWM qua jumper và điện trở gate để đóng ngắt MOSFET;
   dòng buzzer đi qua transistor thay vì đi qua chân MCU.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Các đường LED, nút và buzzer đã được kiểm tra trên netlist; sheet
   không có mục ERC ở lần kiểm tra hiện tại. Cần xác nhận loại buzzer,
   khả năng MOSFET dẫn với gate 3,3 V, đo dòng tải và kiểm tra chống
   dội. Nếu loại buzzer có tính cảm, cần đánh giá thêm bảo vệ khi ngắt.”

## 5. Cảm biến và hiển thị — sensor.kicad_sch

1. **Cần giải quyết việc gì?**

   “Khối cung cấp dữ liệu thực để học đọc xung GPIO, giao tiếp I²C và
   hiển thị. Nó tạo đường thực hành từ cảm biến đến xử lý và giao
   diện quan sát ngay trên board.”

2. **Linh kiện nào thực hiện?**

   “AM2302 thuộc họ DHT22 đo nhiệt độ, độ ẩm; APDS-9306-065 là cảm
   biến ánh sáng số; OLED là màn hình giao tiếp I²C. Các đường dữ
   liệu có pull-up và các linh kiện có tụ bypass nguồn.”

3. **Tín hiệu đi như thế nào?**

   “AM2302 trao đổi các xung dữ liệu với PD3. Cảm biến ánh sáng và
   OLED dùng chung I²C1: PB8 là SCL, PB9 là SDA. MCU đọc cảm biến,
   xử lý rồi gửi dữ liệu hiển thị. Header I²C cho phép nối thêm thiết
   bị nếu mức điện áp và địa chỉ phù hợp. Thiết kế dùng cảm biến
   ánh sáng số; quang trở thụ động tự nó không giao tiếp I²C.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “SCL/SDA đã sửa đúng về PB8/PB9 và sheet hiện không có mục ERC.
   Cần kiểm tra địa chỉ I²C, pinout module mua thực tế, điện trở
   pull-up hiệu dụng và thời gian lên của bus. Pinout AM2302 phải
   khớp linh kiện thực; chưa có dữ liệu đo cảm biến hoặc kiểm chứng
   độ chính xác và hiển thị trên phần cứng.”

## 6. ADC nội, DAC và timer — analog_timer.kicad_sch

1. **Cần giải quyết việc gì?**

   “Khối đưa các chức năng analog và timer của MCU ra đầu nối để đo
   điện áp, tạo điện áp, tạo PWM, đọc encoder và đo độ rộng xung.”

2. **Linh kiện nào thực hiện?**

   “ADC, DAC và timer nằm trong F407. Các header đưa tín hiệu ra ngoài;
   ADC có điện trở 1 kΩ và tụ 10 nF, PWM có điện trở nối tiếp 47 Ω,
   và hai đầu vào encoder có điện trở kéo lên.”

3. **Tín hiệu đi như thế nào?**

   “Ba tín hiệu ADC từ header đi qua bộ lọc tới PA0, PC0 và PB1.
   Hai đầu ra DAC từ PA4 và PA5 đi ra header. TIM1 xuất bốn kênh PWM
   trên PE9, PE11, PE13 và PE14. Encoder vào PD12/PD13 của TIM4;
   tín hiệu đo xung vào PA3 của TIM2. Bốn kênh TIM1 chỉnh duty riêng,
   nhưng chia sẻ cùng chu kỳ nền.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Netlist đã xác nhận bốn PWM tách riêng và các chân analog/timer
   đúng mapping. Cần tính thời gian lấy mẫu ADC theo trở kháng nguồn
   và bộ lọc, kiểm tra dòng tải DAC, mức tín hiệu ngoài và bảo vệ đầu
   vào. Chưa có kết quả đo sai số ADC/DAC hoặc dạng sóng timer.”

## 7. ADC độ phân giải cao — precision_adc.kicad_sch

1. **Cần giải quyết việc gì?**

   “Khối phục vụ bài học đo tín hiệu analog cần độ phân giải cao,
   đọc dữ liệu theo tín hiệu báo sẵn sàng và đánh giá nhiễu. Nó cho
   phép so sánh ADC ngoài với ADC tích hợp trong MCU.”

2. **Linh kiện nào thực hiện?**

   “ADS1220 là ADC delta-sigma 24 bit, có PGA và nguồn tham chiếu nội
   2,048 V. Bốn điện trở 1 kΩ cùng hai tụ vi sai 10 nF lọc đầu vào;
   nhánh AVDD có FB2 và tụ bypass. 24 bit là độ phân giải danh định,
   không có nghĩa độ chính xác thực đạt 24 bit.”

3. **Tín hiệu đi như thế nào?**

   “Tín hiệu từ J13 qua điện trở vào AIN0 đến AIN3. MCU chọn đầu
   vào và cấu hình ADC qua SPI2: PB10/SCK, PC3/MOSI, PC2/MISO.
   PE10 là CS và PD6 nhận DRDY. Khi DRDY báo dữ liệu sẵn sàng,
   MCU có thể đọc kết quả rồi đưa vào buffer. Khối dùng clock nội
   và có thể cấu hình dùng reference nội.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “CS đã nối về PE10 và đủ bốn nhánh lọc được xác nhận trong netlist.
   Cần chọn mã FB2, kiểm tra dải điện áp từng chân, common-mode và
   điều kiện sử dụng PGA. ADS1220 dùng SPI mode 1, khác cấu hình
   MCP2515; firmware phải quản lý bus và CS đúng. Chưa đo offset,
   nhiễu, độ tuyến tính hoặc độ phân giải hiệu dụng.”

## 8. Lưu trữ microSD — storage_sdio.kicad_sch

1. **Cần giải quyết việc gì?**

   “Khối dùng để lưu dữ liệu, thực hành hệ thống file, DMA, quản lý
   buffer và xử lý lỗi ghi. Nó giúp tách thời điểm thu dữ liệu khỏi
   thời điểm thẻ thực sự ghi xong.”

2. **Linh kiện nào thực hiện?**

   “Khe microSD J5 nối tới SDIO của MCU. Các điện trở kéo lên hỗ trợ
   trạng thái của CMD và đường dữ liệu, còn tụ nguồn hỗ trợ nhu cầu
   dòng thay đổi khi thẻ hoạt động.”

3. **Tín hiệu đi như thế nào?**

   “PC8 đến PC11 là D0 đến D3, PC12 là CLK và PD2 là CMD. Trong luồng
   logging dự kiến, dữ liệu cảm biến đi vào buffer RAM, sau đó task
   lưu trữ ghi thành khối qua SDIO. DMA chuyển dữ liệu giữa RAM và
   ngoại vi; FatFs và chính sách đồng bộ file thuộc phần firmware.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Đã đối chiếu các đường SDIO và chân socket trên netlist. Socket
   hiện chưa cung cấp đường card-detect về PD7 theo kế hoạch ban đầu.
   Chưa thử tốc độ ghi, thời gian ghi xấu nhất, tháo thẻ hoặc mất
   nguồn khi ghi. Buffer DMA phải nằm ở vùng RAM mà DMA truy cập được,
   không đặt vào CCM của F407.”

## 9. Cổng CAN thứ nhất — controller nội MCU

1. **Cần giải quyết việc gì?**

   “Cổng này phục vụ truyền thông CAN và mô phỏng một nút ECU. MCU
   xử lý khung CAN, còn transceiver chuyển mức logic thành tín hiệu
   điện vi sai trên bus.”

2. **Linh kiện nào thực hiện?**

   “Controller CAN1 nằm trong F407. U11 SN65HVD230 là transceiver,
   D6 bảo vệ đường bus, J6 là đầu nối CANH/CANL/GND. Điện trở 120 Ω
   có jumper để chỉ bật termination khi board nằm ở đầu cuối bus.”

3. **Tín hiệu đi như thế nào?**

   “CAN1_TX từ PD1 đi vào chân D của transceiver rồi ra CANH/CANL.
   Chiều nhận đi từ CANH/CANL qua chân R về PD0/CAN1_RX. Chân RS
   được cấu hình qua jumper và PE8; nối RS xuống GND chọn chế độ
   tốc độ cao. CAN là đường truyền hai chiều, còn TX/RX là các
   tín hiệu logic giữa controller và transceiver.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Đã kiểm tra TX/RX, bảo vệ, termination và cách nối jumper RS.
   Chưa chứng minh giao tiếp trên bus thực. Cần cấu hình bit timing,
   có nút CAN khác để thử ACK, và đo CANH/CANL; tiếp tục thử lỗi,
   bus-off và phục hồi. Đây là Classical CAN, không phải CAN FD.”

## 10. Cổng CAN thứ hai — controller ngoài qua SPI

1. **Cần giải quyết việc gì?**

   “Cổng thứ hai tạo một bus vật lý độc lập và bài học viết driver
   cho controller ngoài. Nó cũng tạo nền tảng thử chuyển tiếp dữ
   liệu giữa hai mạng CAN bằng phần mềm gateway.”

2. **Linh kiện nào thực hiện?**

   “MCP2515 đảm nhiệm controller CAN ngoài, dùng thạch anh Y3 16 MHz.
   SN65HVD230 thứ hai đảm nhiệm lớp vật lý. Cổng có bảo vệ bus,
   đầu nối và điện trở termination riêng.”

3. **Tín hiệu đi như thế nào?**

   “F407 điều khiển MCP2515 qua SPI2, với PE7 là CS, PD4 nhận INT
   và PD5 điều khiển reset. MCP2515 trao đổi TXCAN/RXCAN với U13,
   sau đó U13 nối tới CANH/CANL của J7. PE12 tham gia điều khiển RS.
   Cổng này không dùng peripheral CAN2 nội của F407.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Netlist đã kiểm tra các đường SPI, điều khiển và transceiver
   riêng. Cần viết driver, kiểm tra clock, bit timing và trạng thái
   interrupt. MCP2515 dùng SPI mode 0 hoặc 3; khi dùng chung với
   ADS1220 cần chọn CS, đổi mode khi bus rảnh và khóa bus phù hợp.
   Chưa có kết quả thử gateway, tải bus hay độ trễ chuyển tiếp.”

## 11. Ethernet — ethernet.kicad_sch

1. **Cần giải quyết việc gì?**

   “Ethernet phục vụ trao đổi dữ liệu với máy tính Linux và học mạng
   nhúng. F407 có MAC nhưng vẫn cần PHY ngoài và phần biến áp cách
   ly để nối ra cáp mạng.”

2. **Linh kiện nào thực hiện?**

   “U15 LAN8720A là PHY; J14 Amphenol là MagJack có biến áp, mạng
   termination phía cáp và LED tích hợp. Y4 là thạch anh 25 MHz.
   Các điện trở strap chọn chế độ khởi động; điện trở 49,9 Ω và
   RBIAS 12,1 kΩ phục vụ phần analog của PHY.”

3. **Tín hiệu đi như thế nào?**

   “MAC của F407 trao đổi dữ liệu với PHY qua RMII: TXD0/TXD1/TX_EN
   theo chiều phát, RXD0/RXD1/CRS_DV theo chiều nhận. MDIO/MDC dùng
   để cấu hình và đọc trạng thái, không mang payload Ethernet.
   PHY dùng thạch anh 25 MHz để tạo REFCLKO 50 MHz đưa về PA1.
   Dữ liệu từ PHY qua biến áp trong MagJack rồi ra cáp mạng.
   Chân 14 đã dùng xuất clock nên không đồng thời làm interrupt.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Ở commit cbc661b, netlist xác nhận RXD0/RXD1 độc lập và đủ mười
   đường RMII, management và reset giữa PHY với MCU. Sheet Ethernet
   có 0 lỗi ERC và một cảnh báo do tên ETH_AVDD cùng net với 3,3 V.
   Cần chốt mã thạch anh để tính lại C53/C54, làm layout đúng và đo
   nguồn lõi, clock, reset, PHY ID và link. Ping, TCP/UDP và thông
   lượng đều là các phép thử tiếp theo, chưa phải kết quả hiện có.”

## 12. Mở rộng, USB target và các mục chưa hoàn thiện

1. **Cần giải quyết việc gì?**

   “Nhóm cuối giúp mở rộng kit và sử dụng nó thuận tiện trong phòng
   thực hành: debug target bằng thiết bị ngoài, xuất console UART,
   thêm thiết bị SPI và nối USB trực tiếp với F407.”

2. **Linh kiện nào thực hiện?**

   “Các phần dự kiến gồm header SWD/SWO cho target, header UART,
   header SPI, connector USB target và bảo vệ USB. LSE và nguồn
   dự phòng RTC cũng thuộc danh sách cần hoàn thiện.”

3. **Tín hiệu đi như thế nào?**

   “SWD target dùng PA13/PA14, reset và PB3/SWO. UART console dự
   kiến dùng PB6/PB7. SPI mở rộng dùng chung SPI2 với một CS riêng.
   USB target dự kiến dùng PA11/PA12 và đường phát hiện VBUS được
   thiết kế phù hợp. USB debugger đang có chỉ nối vào F103; nó
   không thay thế USB target của F407.”

4. **Đã xác minh được gì và còn thiếu gì?**

   “Đây là các phần đang có kế hoạch phân bổ chân, chưa được triển
   khai đầy đủ trong schematic hiện tại. Cần hoàn thiện đầu nối,
   bảo vệ, pinout và firmware rồi mới kiểm thử. Trước khi vẽ tiếp
   phải thống nhất tên net CS của header SPI giữa tài liệu và sơ đồ.”

## Kết thúc

“Thiết kế tạo một chuỗi bài học từ nhập xuất số và cảm biến, đến thu
thập dữ liệu, lưu trữ và truyền thông. Các khối được chia sheet để
quản lý và nối với nhau bằng các net đã phân bổ cho MCU.

Kết quả hiện có là schematic cùng các kiểm tra kết nối, chưa phải
chứng minh phần cứng đã hoạt động. Toàn project còn 74 mục ERC ở lần
kiểm tra này. Bước tiếp theo là hoàn thiện các phần còn thiếu, xử lý
các mục ERC theo nguyên nhân, chốt linh kiện và layout, rồi thử từng
khối theo thứ tự nguồn, core/debug, ngoại vi và truyền thông.”

## Cách hiểu và trả lời khi được hỏi

| Khái niệm | Cách giải thích ngắn |
| --- | --- |
| Schematic | Mô tả linh kiện và các kết nối điện dự kiến. |
| Netlist | Danh sách chân thuộc cùng một mạng điện; giúp phát hiện nối nhầm hoặc thiếu đường. |
| ERC | Kiểm tra quy tắc theo kiểu chân và kết nối; không chứng minh điện áp, timing hay tín hiệu đạt yêu cầu. |
| Controller và transceiver CAN | Controller xử lý khung, arbitration và lỗi giao thức; transceiver chuyển tín hiệu logic sang bus vật lý. |
| MAC và PHY Ethernet | MAC xử lý lớp liên kết dữ liệu; PHY xử lý tín hiệu vật lý. |
| Strap | Điện trở đặt mức cấu hình tại thời điểm reset; chân có thể chuyển sang chức năng khác sau đó. |
| DMA | Chuyển dữ liệu giữa ngoại vi và vùng nhớ được hỗ trợ; vẫn cần cấu hình và xử lý hoàn tất/lỗi. |
| FreeRTOS | Tổ chức task và đồng bộ tài nguyên; không tự giải quyết tranh chấp bus hoặc mất dữ liệu. |
| Buffer | Vùng nhớ đệm để hấp thụ chênh lệch tốc độ; dung lượng cần dựa trên tốc độ dữ liệu và thời gian phục vụ xấu nhất. |

Nếu chưa đo, nên nói: “Em đã xác nhận đường nối trong netlist; phép đo
cần thực hiện là …”. Nếu chưa có firmware, nên nói: “Phần cứng tạo điều
kiện cho chức năng này; firmware và phép thử vẫn cần thực hiện”.

Không dùng “nguồn ổn định”, “ADC chính xác 24 bit”, “DMA không mất dữ
liệu”, “đã chạy FreeRTOS”, “Ethernet đạt 100 Mbps” hoặc “debugger hỗ trợ
đủ SWO/VCP” như kết quả thực nghiệm khi chưa có bằng chứng tương ứng.
