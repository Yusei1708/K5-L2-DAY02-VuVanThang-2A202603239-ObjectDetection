# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe ô tô con chở người cá nhân/gia đình (sedan, hatchback, SUV, crossover, coupe, taxi) và xe bán tải (pickup) có cabin nhỏ. | Có nắp capo và cốp tách biệt hoặc dáng SUV/hatchback thấp; khác với `van` ở chiều cao thân xe và thiết kế đầu xe dài hơn; khác `truck` vì không có thùng hàng tải trọng lớn. |
| `truck` | Xe tải chở hàng chuyên dụng (xe tải thùng kín, thùng lửng, thùng mui bạt, xe ben chở vật liệu, xe đầu kéo/container). | Có cabin đầu kéo tách biệt với thùng hàng lớn phía sau hoặc gầm cao, bánh lớn chở hàng nặng (như xe tải ben màu đỏ ở `drive_040.jpg`); không tính xe bán tải nhỏ (`car`) hay xe van chở hàng thân liền khối (`van`). |
| `bus` | Xe buýt công cộng, xe buýt khớp nối (articulated bus), xe khách cỡ lớn hoặc minibus chở khách có khoang hành khách dài với dãy cửa sổ liên tục. | Thân xe dài, cao, đầu phẳng, nhiều ô cửa kính dọc thân và có biển tuyến/cửa lên xuống cho hành khách (như xe buýt khớp nối vàng-trắng ở `drive_040.jpg` hoặc xe buýt trắng ở `drive_025.jpg`). |
| `van` | Xe van/minivan đa dụng chở hàng nhẹ hoặc chở người (MPV cỡ vừa/nhỏ dạng hộp như xe tải van nhỏ / microvan, transit van). | Thân xe dạng khối hộp liền mạch từ đầu tới đuôi, mũi xe rất ngắn hoặc phẳng, kích thước nhỏ và ngắn hơn hẳn `bus` nhưng trần cao và vuông vức hơn `car` (sedan/SUV). |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** 4 cạnh của bounding box (trái, phải, trên, dưới) phải tiếp xúc sát với điểm ngoài cùng của phần thân xe và bánh xe **nhìn thấy được** trong ảnh; không bao gồm bóng đổ của xe trên mặt đường, ánh đèn pha loang ra đường, hay khoảng trống nền đường/bầu trời xung quanh. Mỗi phương tiện hợp lệ là 1 box độc lập.
- **Vật thể bị che khuất:** Nếu xe bị che khuất một phần bởi cột đèn, cây cối dải phân cách hoặc xe khác (occlusion) nhưng phần nhìn thấy được vẫn đủ rõ để xác định chắc chắn lớp (`car`, `truck`, `bus`, `van`), vẫn vẽ 1 bounding box ôm sát phần diện tích nhìn thấy của xe (hoặc toàn bộ khung xe liên tục nếu chỉ bị vật mảnh như cột đèn cắt ngang ở giữa mà vẫn thấy rõ cả đầu và đuôi xe).
- **Vật thể bị cắt bởi rìa ảnh:** Khi xe đi vào hoặc đi ra khỏi khung hình (truncation), vẽ bounding box bao quanh phần thân xe nằm trong ảnh và kéo cạnh box chạm đúng mép ảnh (`x=0`, `y=0`, `x=1`, hoặc `y=1`); tuyệt đối không suy đoán phần xe nằm ngoài khung hình.
- **Vật thể rất nhỏ/mơ hồ:** Các phương tiện ở quá xa phía chân trời chỉ còn vài pixel bị nhòe/lẫn vào nền hoặc bị che khuất >80% không thể phân biệt rõ loại xe thì không tự đoán gán bừa (ghi chú lại). Không gán nhãn cho xe máy/mô tô, xe đạp, xe ba gác, người đi bộ, biển báo hay hình phản chiếu.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001.jpg` | Chiếc xe màu trắng ở làn bên phải bị cột đèn đường dọc che cắt ngang chính giữa thân xe; đồng thời có dãy xe ô tô đang đỗ tĩnh dọc lề đường bên trái. | Vẽ 1 box `car` (`0`) bao trọn từ mũi tới đuôi chiếc xe trắng bị cột đèn chắn ngang; đồng thời gán nhãn `car` (`0`) cho từng chiếc xe đang đỗ dọc lề trái còn nhìn rõ hình dáng. | Cột đèn là vật cản mảnh tiền cảnh nhưng hai đầu chiếc xe trắng vẫn lộ rõ là cùng 1 chiếc SUV (`car`), không được tách thành 2 box rời. Các xe đang đỗ vẫn thuộc lớp vật thể `car` vì schema phân loại theo **loại phương tiện** chứ không phân biệt trạng thái đang chạy hay đang đỗ. |
| `drive_010.jpg` | Cảnh giao thông ban đêm có người đi bộ đứng gần dải phân cách, người điều khiển xe hai bánh (xe máy/xe đạp điện) trên lòng đường, và chiếc xe màu đỏ ở mép trái bị xe khác che một phần. | Không gán nhãn cho người đi bộ và xe hai bánh; chỉ gán `car` (`0`) cho chiếc xe màu đỏ ở rìa trái trên phần thân nhìn thấy được và các ô tô con khác trên đường. | Theo schema bắt buộc, phạm vi gán nhãn chỉ gồm 4 lớp (`car`, `truck`, `bus`, `van`); người đi bộ và mô tô/xe máy nằm ngoài scope nên phải bỏ qua để tránh tạo False Positive (FP). |
| `drive_025.jpg` | Chiếc xe con màu tối ở làn bên phải dải phân cách bị hàng cây xanh che khuất toàn bộ phần thân dưới (chỉ lộ phần nóc, kính và đuôi trên), và chiếc xe trắng dáng hộp nhỏ chạy song song với sedan trắng ở làn trên. | Vẫn gán 1 box `car` (`0`) bám sát phần thân trên nhìn thấy được của chiếc xe tối màu sau bụi cây; phân loại chiếc xe trắng dáng vuông nhỏ phía làn bên kia theo đúng đặc trưng thân xe (`van` / `car`) và vẽ 2 box tách biệt cho 2 xe trắng đi gần nhau. | Theo quy tắc occlusion, xe bị bụi cây dải phân cách che một phần nhưng vẫn nhận diện rõ là ô tô con (`car`) nên bắt buộc phải có box quanh phần nhìn thấy; hai xe đi sát nhau phải tách thành 2 box riêng biệt, không gộp chung. |
| `drive_040.jpg` | Ngã tư đông đúc xuất hiện đồng thời xe buýt khớp nối rất dài (vàng-trắng), xe tải ben màu đỏ chở đất, các xe minivan thân hộp màu bạc/trắng và nhiều người đi xe máy. | Gán toàn bộ thân xe buýt khớp nối thành 1 box `bus` (`2`), xe ben đỏ là `truck` (`1`), các xe minivan thân hộp đầu ngắn ở giữa ngã tư và bên trái là `van` (`3`), các xe sedan/SUV là `car` (`0`), bỏ qua toàn bộ xe máy. | Xe buýt khớp nối tuy có 2 khoang nối bằng khớp mềm nhưng là 1 phương tiện `bus` duy nhất; xe minivan thân hộp vuông đầu phẳng thuộc `van` (`3`) cần tách biệt rõ với `car` (`0`) và `bus` (`2`). |
