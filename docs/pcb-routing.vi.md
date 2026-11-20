# Hướng dẫn bố trí và đi dây STM32F407VET6 Education Kit

PCB hiện có 4 lớp, dày 1,6 mm. Stackup trong KiCad khai báo đồng 35 µm trên 4 lớp. Đây là dữ liệu thiết kế để đối chiếu với stackup thực tế của nhà gia công. Bản đang có 1.574 đoạn mạch và 195 via, nhưng chưa có vùng đổ đồng. Cần đồng bộ khối Ethernet và xử lý DRC trước khi xuất Gerber.

## Đặt nhãn trên schematic

| Nhóm | Ví dụ trong project | Cách dùng |
|---|---|---|
| Tín hiệu đi giữa các trang | `I2C1_SCL`, `SDIO_CK`, `TIM1_CH1`, `GPIO_BTN1_N` | Đặt global label ở đầu một đoạn dây ngắn. Dùng cùng tên ở MCU và ngoại vi. |
| Tín hiệu chỉ dùng trong một trang | `CAN1_P`, `CAN1_N`, `MDI_TX_P`, `MDI_TX_N` | Dùng local label. Giữ đường nối trực tiếp từ IC đến điện trở, tụ hoặc TVS để dễ thấy nguyên lý. |
| Cặp vi sai | `DBG_USB_D_P/N`, `DBG_MCU_USB_P/N`, `MDI_RX_P/N`, `CAN_EXT_P/N` | Hai nhãn có cùng phần tên gốc và hậu tố `_P` / `_N` để router nhận diện. CANH tương ứng P, CANL tương ứng N. |
| Tín hiệu tác động mức thấp | `GPIO_BTN1_N`, `ADC_EXT_CS_N`, `ETH_RESET_N` | Hậu tố `_N` chỉ cực tính. Khai báo cặp vi sai chỉ khi thực sự có cả P và N cùng chức năng. |

Vẽ theo luồng tín hiệu. Chỉ dùng nhãn tại nơi cần qua trang, qua cụm hoặc tránh đường vòng. Giữ nguồn, linh kiện lọc và đường hồi dòng gần linh kiện trên hình. Với U1, sơ đồ sắp chân theo port để tra logic. Khi đặt khối trên PCB, dùng số chân và vị trí pad thật của LQFP100 để chọn phía thoát dây.

Ví dụ, SDIO chạy ra từ PC8/PC9 ở chân 65/66 và PC10/PC11/PC12/PD2 ở chân 78–80/83. Thử đặt và xoay socket về phía góc giữa hai cạnh có các nhóm pad đó, rồi đánh giá lối đi cho CLK/CMD/data trước khi khóa vị trí.

## Bề rộng dây, khoảng hở và stackup

**Bề rộng đường mạch** là chiều rộng nhìn từ trên xuống. **Độ dày đồng** là chiều dày lớp đồng. Với cùng chiều rộng, đồng dày hơn giảm điện trở. Với cặp tốc độ cao, bề rộng và khe hở còn phụ thuộc lớp tham chiếu, độ dày điện môi và hằng số điện môi của stackup.

| Nhóm | Bề rộng hoặc mục tiêu | Áp dụng |
|---|---|---|
| GPIO, LED, nút, I2C, SWD, SPI, SDIO/RMII | 0,20 mm là preset hiện tại, clearance 0,20 mm | Dùng để bắt đầu fan-out LQFP100. Với đường có yêu cầu trở kháng, tính lại theo stackup và tốc độ. |
| Nguồn nhánh | `Power_Local` 0,35 mm | 3V3_DBG và các nguồn analog sau bead. Xác định theo dòng của nhánh. |
| Nguồn trục | `Power_Trunk` 0,60 mm | 12 V, 5 V và 3,3 V hệ thống. Với đoạn dài hoặc tải lớn, tính sụt áp và nhiệt, cân nhắc polygon đồng. |
| USB 2.0 FS | Mục tiêu **90 Ω vi sai** | Chốt width/gap theo stackup nhà gia công. Route hai nhánh đồng hành qua ESD, giảm stub và tránh đổi lớp. |
| Ethernet PHY ↔ magnetics | Mục tiêu **100 Ω vi sai** | Dùng router cặp cho `MDI_TX_P/N` và `MDI_RX_P/N`. RMII giữa U1 và U15 là các đường đơn. |
| CAN | Giữ CANH/CANL gần nhau và cân đối | Kiểm tra trở kháng và chiều dài theo từng đoạn. Điện trở kết thúc 120 Ω mắc giữa CANH/CANL khi KIT ở đầu cuối bus. |

Preset là kích thước khi router dùng giá trị net class. Board vẫn dùng giới hạn toàn cục 0,20 mm và các trace cũ giữ kích thước đã route. Ví dụ với đường đồng ngoài 35 µm dài 50 mm, rộng 0,60 mm có điện trở khoảng 41 mΩ ở nhiệt độ phòng. Dòng 1 A gây sụt khoảng 41 mV trên riêng đoạn đó. Tính cả đường hồi, via, đầu nối và độ tăng nhiệt theo yêu cầu của nguồn.

Nếu dùng 4 lớp, ưu tiên **F.Cu** cho linh kiện và tín hiệu chính, **In1.Cu** làm mặt phẳng GND liền, **In2.Cu** cho nguồn và đường chậm có kiểm soát, **B.Cu** cho tín hiệu cần thoát lớp. Khi route trên B.Cu, kiểm tra mặt tham chiếu gần nhất là In2.Cu và tránh để tín hiệu nhanh băng qua khe giữa các vùng nguồn. Khi đổi lớp qua một mặt tham chiếu GND khác, bố trí đường hồi bằng via GND gần via tín hiệu. Chốt stackup trước khi cấu hình trở kháng. [KiCad PCB Editor](https://docs.kicad.org/10.0/en/pcbnew/pcbnew.html), [cách nhập stackup vào bộ tính trở kháng](https://jlcpcb.com/help/article/user-guide-to-the-jlcpcb-impedance-calculator)

## Thứ tự đặt linh kiện và route

1. **Outline và cơ khí.** Khóa lỗ bắt vít và vị trí đầu nối theo vỏ. Đặt USB-C, jack DC, microSD, CAN và RJ45 sao cho cắm/rút được. Đặt ESD sát cổng và nối GND bằng đường ngắn.
2. **Đồng bộ Ethernet trước khi khóa chỗ.** Dùng Update PCB from Schematic (F8) trên bản làm việc, đọc danh sách thay đổi trước khi áp dụng. PCB hiện thiếu 26 footprint trang Ethernet, gồm U15 và J15. JP11 và JP15 còn trên board nhưng không xuất hiện trong netlist schematic hiện tại. Kiểm tra UUID và footprint ở bảng cập nhật trước khi xử lý hai jumper. Chừa hành lang ngắn U1 → U15 → J15 và mặt phẳng liên tục dưới vùng RMII. [Microchip AN186](https://www.microchip.com/en-us/application-notes/an186)
3. **Nguồn và MCU.** Đặt tụ VDD/VCAP sát đúng cặp chân, tụ bootstrap sát chân BST/SW và giữ vòng dòng VIN–IC–GND của buck nhỏ. Dùng đồng rộng ở nơi mang dòng. Giữ vùng SW gọn và xa ADC, thạch anh, USB. Đặt thạch anh sát chân dao động và tránh đi tín hiệu nhanh qua vùng đó. [ST AN4488](https://www.st.com/resource/en/application_note/an4488-getting-started-with-stm32f4xxxx-mcu-hardware-development-stmicroelectronics.pdf)
4. **Tín hiệu cần ưu tiên.** Route USB, Ethernet MDI/RMII, SDIO và clock trước. Với RMII, ưu tiên REF_CLK 50 MHz ngắn và có GND liền bên dưới. Với SDIO, nhóm CK/CMD/D0–D3, hạn chế via và rà chênh lệch chiều dài theo tốc độ thực tế. Với USB, bám cấu hình cặp theo stackup. [ST AN4879](https://www.st.com/content/st_com/en/technical-documents/an4879-introduction-to-usb-hardware-and-pcb-guidelines-using-stm32-mcus-stmicroelectronics.html)
5. **Tín hiệu thường và analog.** Dùng highlight net để kiểm tra trực quan mỗi cụm. Đặt bộ lọc đầu vào gần ADC, giữ đường analog xa cuộn cảm và SW. Chọn mặt phẳng GND liên tục rồi tách vùng bằng placement và đường hồi. Để dư cổng đo cho rail, NRST, SWD và REF_CLK nếu layout cho phép.
6. **Kiểm tra sau khi đổ đồng.** Tạo plane GND, kiểm tra thermal relief và vùng đồng mồ côi, refill bằng B rồi chạy ERC/DRC. Rà các nhóm lỗi clearance, lỗ khoan, đường/via treo và silkscreen. Đối chiếu schematic–PCB sau F8 và xem Gerber trước khi đặt PCB.

## Kết quả kiểm tra sau khi chỉnh nhãn

Netlist đối chiếu theo từng reference và số chân xác nhận 185 linh kiện và 210 net giữ nguyên kết nối. Ba vị trí giao nhau giữa các dây trên schematic được loại bỏ. ERC giảm từ 70 xuống 64 mục. DRC hình học vẫn báo 74 mục. Mục `0 unconnected` của PCB chỉ tính footprint đang đặt, vì vậy dùng thêm đối chiếu với schematic để phát hiện các footprint Ethernet còn thiếu.
