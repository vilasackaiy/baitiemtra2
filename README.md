# baikiemtra2
Họ và tên: Panyasack Vilasack

MSSV: LAOS225001

Lớp: K59KMT.K01

Phần 1: Thiết kế và Khởi tạo Cấu trúc Dữ liệu

1.1. Mô tả logic bài toán

Em chọn chủ đề Quản lý hệ thống nạp tiền Game (Game Top-up Management). Đây là một hệ thống thực tế cho phép người dùng đăng ký tài khoản, xem các gói nạp và thực hiện giao dịch nạp tiền vào tài khoản game.


1.2. Mã SQL khởi tạo cấu trúc dữ liệu SQL

-- Tạo Database theo yêu cầu [Tên dự án]_[MaSV]

    CREATE DATABASE [QuanLyNapGame_LAOS225001];
    
    GO

    USE [QuanLyNapGame_LAOS225001];
    
    GO
<img width="960" height="540" alt="1" src="https://github.com/user-attachments/assets/09880e67-21a7-423f-881a-66017674b0a2" />

-- 1. Tạo bảng Người dùng (NguoiDung)

    CREATE TABLE [NguoiDung] (

    [MaND] INT PRIMARY KEY IDENTITY(1,1), -- Khóa chính tự tăng
    
    [TenDangNhap] NVARCHAR(50) NOT NULL UNIQUE, -- Tên đăng nhập là duy nhất
    
    [SoDu] MONEY DEFAULT 0 CHECK ([SoDu] >= 0) -- Ràng buộc CK: Số dư không được âm

    );

<img width="960" height="540" alt="bang ng dung-1" src="https://github.com/user-attachments/assets/71c01bf2-13b3-49c2-a893-efa9b1706d1d" />


-- 2. Tạo bảng Gói nạp (GoiNap)

    CREATE TABLE [GoiNap] (
    
    [MaGoi] INT PRIMARY KEY IDENTITY(1,1), -- Khóa chính
    
    [TenGoi] NVARCHAR(100) NOT NULL,
    
    [GiaTien] MONEY NOT NULL CHECK ([GiaTien] > 0) -- Ràng buộc CK: Giá tiền phải lớn hơn 0
    
    );

  <img width="960" height="540" alt="bang goi nap-1" src="https://github.com/user-attachments/assets/47a3cded-08a8-4285-b948-131adb5f3729" />


-- 3. Tạo bảng Giao dịch (GiaoDich)

    CREATE TABLE [GiaoDich] (
    
    [MaGD] INT PRIMARY KEY IDENTITY(1,1),
    
    [MaND] INT FOREIGN KEY REFERENCES [NguoiDung]([MaND]), -- Khóa ngoại liên kết bảng NguoiDung
    
    [MaGoi] INT FOREIGN KEY REFERENCES [GoiNap]([MaGoi]), -- Khóa ngoại liên kết bảng GoiNap
    
    [NgayGD] DATETIME DEFAULT GETDATE() -- Mặc định lấy ngày hiện tại
    
    );

<img width="960" height="540" alt="bang giao dich -1" src="https://github.com/user-attachments/assets/a783a27f-28e9-41e9-bfaa-475a4406757a" />

Phần 2: Xây dựng Function (Kiến thức 8, 9)

2.1. Các loại hàm có sẵn (Built-in Functions) trong SQL Server

Trong SQL Server, các hàm build-in được chia thành nhiều loại chính nhằm hỗ trợ xử lý dữ liệu nhanh chóng:

    Hàm toán học (Mathematical Functions): Như ABS, ROUND, CEILING.

    Hàm chuỗi (String Functions): Như LEN, UPPER, LOWER, SUBSTRING.

    Hàm ngày tháng (Date and Time Functions): Như GETDATE(), YEAR(), DATEDIFF.

    Hàm tổng hợp (Aggregate Functions): Như SUM, AVG, COUNT, MAX, MIN.

Một số System Functions đặc sắc:

COALESCE(val1, val2, ...): Trả về giá trị đầu tiên khác NULL trong danh sách. Rất hữu ích khi xử lý dữ liệu trống.

Khai thác: SELECT TenGoi, COALESCE(GiaTien, 0) FROM [GoiNap];

DATEDIFF(datepart, start, end): Tính toán khoảng cách giữa hai mốc thời gian.

Khai thác: SELECT DATEDIFF(day, '2024-01-01', GETDATE()) AS SoNgay;

2.2. Hàm do người dùng tự viết (User-Defined Functions - UDF)

Mục đích: Dùng để đóng gói các logic nghiệp vụ phức tạp hoặc các công thức tính toán lặp đi lặp lại, giúp mã nguồn gọn gàng và dễ tái sử dụng.

Phân loại:

Scalar Function (Hàm đơn trị): Trả về một giá trị duy nhất.

Inline Table-Valued Function (Hàm bảng nội tuyến): Trả về một tập dữ liệu (bảng) thông qua một câu lệnh SELECT.

Multi-statement Table-Valued Function (Hàm bảng đa câu lệnh): Trả về một bảng nhưng chứa nhiều bước xử lý phức tạp bên trong.

Tại sao cần tự viết: Để giải quyết các bài toán mang tính đặc thù của dự án mà các hàm có sẵn không hỗ trợ (ví dụ: công thức tính thuế hoặc phân loại riêng).

2.3. Thực thi các hàm tự định nghĩa
1. Scalar Function (Hàm trả về một giá trị)

Yêu cầu: Tính thuế VAT (10%) cho các gói nạp dựa trên giá tiền.

Mã SQL:

      CREATE FUNCTION fn_TinhThueVat(@GiaTien MONEY)
      
      RETURNS MONEY
    
    AS
    
    BEGIN
    RETURN @GiaTien * 0.1;
    END;

    Khai thác:

    SELECT TenGoi, GiaTien, dbo.fn_TinhThueVat(GiaTien) AS TienThue 
    FROM [GoiNap];

<img width="960" height="540" alt="{54169291-EEA6-4AB1-ACDF-9EF5F42F4390}" src="https://github.com/user-attachments/assets/f32a8d2e-c377-49f0-bd58-a7b553996739" />

2. Inline Table-Valued Function (Hàm bảng nội tuyến)

Yêu cầu: Trả về danh sách các gói nạp có giá tiền từ 50,000 trở lên.

Mã SQL:

    CREATE FUNCTION fn_LocGoiNapTheoGia(@GiaToiThieu MONEY)
    RETURNS TABLE
    AS
    RETURN (
    SELECT * FROM [GoiNap] WHERE GiaTien >= @GiaToiThieu
    );

    Khai thác:

    SELECT * FROM dbo.fn_LocGoiNapTheoGia(50000);

<img width="960" height="540" alt="{2D279E2B-2E92-40FB-A764-5617059350FA}" src="https://github.com/user-attachments/assets/58151394-c100-4cb3-a271-ccc222876e36" />

3. Multi-statement Table-Valued Function (Hàm bảng đa câu lệnh)
 Yêu cầu: Xuất danh sách gói nạp kèm theo cột phân loại "Giá Cao" (giá > 100.000) hoặc "Giá Rẻ".

Mã SQL:
   
    CREATE FUNCTION fn_BaoCaoPhanLoaiGoi()
    RETURNS @BangPhanLoai TABLE (Ten NVARCHAR(50), Gia MONEY, PhanLoai NVARCHAR(20))
    AS
    BEGIN
    INSERT INTO @BangPhanLoai
    SELECT TenGoi, GiaTien, 
           CASE WHEN GiaTien > 100000 THEN N'Giá Cao' ELSE N'Giá Rẻ' END
    FROM [GoiNap];
    RETURN;
    END;

    Khai thác:

    SELECT * FROM dbo.fn_BaoCaoPhanLoaiGoi();

<img width="960" height="540" alt="{3C810C60-8B61-46F5-BB51-72E4D5633DF7}" src="https://github.com/user-attachments/assets/633f253e-9305-4cd4-a2e7-cdbfdaebe576" />

Phần 3: Xây dựng Store Procedure (Kiến thức 10)
3.1. Các Store Procedure (SP) có sẵn trong SQL Server

Trong SQL Server, các System Store Procedure giúp quản lý cơ sở dữ liệu và truy xuất thông tin hệ thống một cách nhanh chóng. Các SP này thường bắt đầu bằng tiền tố sp_.

Một số System SP đặc sắc:

sp_help: Cung cấp thông tin chi tiết về các đối tượng trong cơ sở dữ liệu như bảng (cột, kiểu dữ liệu, khóa chính).

Cách dùng: EXEC sp_help '[GoiNap]';

sp_databases: Liệt kê tất cả các cơ sở dữ liệu hiện có trên Server.

Cách dùng: EXEC sp_databases;

3.2. Thực thi các Store Procedure tự định nghĩa
1. SP thực hiện lệnh INSERT có kiểm tra logic

 Yêu cầu: Tạo SP để thêm gói nạp mới, kiểm tra nếu giá tiền nhỏ hơn hoặc bằng 0 thì thông báo lỗi và không cho phép thêm vào bảng.

 Mã SQL:
    
    CREATE PROCEDURE sp_ThemGoiNap
    @TenGoi NVARCHAR(50),
    @Gia MONEY
    AS
    BEGIN
    IF (@Gia <= 0)
    BEGIN
        PRINT N'Giá tiền phải lớn hơn 0!';
    END
    ELSE
    BEGIN
        INSERT INTO [GoiNap] (TenGoi, GiaTien)
        VALUES (@TenGoi, @Gia);
        PRINT N'Thêm gói nạp thành công.';
    END
    END;

    Khai thác:

    EXEC sp_ThemGoiNap N'Gói Kim Cương Siêu Cấp', 1000000;

<img width="960" height="540" alt="{83F536B4-7AA1-4DA6-91C5-D83D38F740BA}" src="https://github.com/user-attachments/assets/7a389c60-b72d-4723-bcbc-184aa9dfaa3d" />


2. SP sử dụng tham số OUTPUT để trả về một giá trị

Yêu cầu: Tạo SP để đếm tổng số lượng gói nạp hiện có trong hệ thống và trả giá trị về thông qua tham số OUTPUT.

Mã SQL:
   
    CREATE PROCEDURE sp_DemSoGoiNap
    @TongSoGoi INT OUTPUT
    AS
    BEGIN
    SELECT @TongSoGoi = COUNT(*) FROM [GoiNap];
    END;

    Khai thác:
   
    DECLARE @SoLuong INT;
    EXEC sp_DemSoGoiNap @TongSoGoi = @SoLuong OUTPUT;
    SELECT @SoLuong AS [TongSoGoiHienCo];

<img width="960" height="540" alt="{115EF997-5458-4D90-95A9-32191FF91B42}" src="https://github.com/user-attachments/assets/301264d3-7035-4c38-9d1f-082fe0e4c37e" />


3. SP trả về tập kết quả (Result set) từ lệnh SELECT Join nhiều bảng

   Yêu cầu: Hiển thị danh sách chi tiết các gói nạp kèm theo thông tin từ bảng liên quan (Ví dụ: Danh mục gói nạp hoặc Lịch sử giao dịch).

 Mã SQL:

    CREATE PROCEDURE sp_DanhSachChiTietGoiNap
    AS
    BEGIN
    -- Lưu ý: Bạn cần thay tên bảng phù hợp với DB thực tế của mình
    SELECT g.TenGoi, g.GiaTien, d.TenDanhMuc
    FROM [GoiNap] g
    JOIN [DanhMucGoi] d ON g.MaDM = d.MaDM;
    END;

    Khai thác:


    EXEC sp_DanhSachChiTietGoiNap;

<img width="960" height="540" alt="{78518DAD-C447-4804-8AFE-9C56A01189D9}" src="https://github.com/user-attachments/assets/bec0df2d-5a62-4dda-8adc-3df3485c2187" />

Phần 4: Trigger và Xử lý logic nghiệp vụ (Kiến thức 11)

4.1. Khái niệm và mục đích của Trigger

Trigger là gì? Là một khối lệnh SQL tự động thực thi khi có sự kiện INSERT, UPDATE hoặc DELETE xảy ra trên một bảng cụ thể.

Mục đích: Dùng để tự động hóa các nghiệp vụ liên quan giữa các bảng, đảm bảo tính toàn vẹn dữ liệu mà các ràng buộc (Constraints) thông thường không xử lý được.

4.2. Viết Trigger tự động cập nhật số dư (Logic thực tế)

 Yêu cầu: Mỗi khi có giao dịch mới (INSERT) vào bảng GiaoDich, hệ thống sẽ tự động trừ số dư tiền của người dùng trong bảng NguoiDung.

    Mã SQL:

    CREATE TRIGGER trg_CapNhatSoDu
    ON [GiaoDich]
    AFTER INSERT
    AS
    BEGIN
    UPDATE [NguoiDung]
    SET [SoDu] = [NguoiDung].[SoDu] - g.[GiaTien]
    FROM [NguoiDung] n
    JOIN inserted i ON n.[MaND] = i.[MaND]
    JOIN [GoiNap] g ON i.[MaGoi] = g.[MaGoi];
    
    PRINT N'Đã tự động cập nhật giảm số dư người dùng.';
    END;

<img width="960" height="540" alt="{6B96B888-80C4-47C2-ADE4-39C48F1E2A6E}" src="https://github.com/user-attachments/assets/b37c730c-56a4-4507-9349-811054f3b0c1" />

4.3. Thử nghiệm Trigger vòng lặp (Recursive Trigger)

Yêu cầu: Thử nghiệm tạo 2 Trigger tác động qua lại lẫn nhau giữa bảng A và bảng B để quan sát hiện tượng.

Bước 1: Đã có Trigger trên bảng GiaoDich cập nhật bảng NguoiDung (ở mục 4.2).

Bước 2: Viết thêm Trigger trên bảng NguoiDung cập nhật ngược lại bảng GiaoDich.

Mã SQL tạo Trigger vòng lặp:


    CREATE TRIGGER trg_NguocLai
    ON [NguoiDung]
    AFTER UPDATE
    AS
    BEGIN
    UPDATE [GiaoDich] SET [NgayGD] = GETDATE() 
    WHERE [MaND] IN (SELECT [MaND] FROM inserted);
    
    PRINT N'Trigger bảng B đang tác động ngược lại bảng A...';
    END;

<img width="960" height="540" alt="{301314EE-3DD5-4317-A162-B1D2DB642848}" src="https://github.com/user-attachments/assets/9dd25c21-e8a7-4f08-b68a-38e47a797470" />


4.4. Quan sát và Giải thích hiện tượng

Thực hiện lệnh kiểm tra:

    INSERT INTO [GiaoDich] (MaND, MaGoi, NgayGD) VALUES (1, 1, GETDATE());


<img width="960" height="540" alt="{B1623F13-6ACF-4503-89E3-AE87CBB389C4}" src="https://github.com/user-attachments/assets/d52a6f0e-a116-4c3c-9b2b-2ba85980c075" />

Kết quả quan sát từ hình ảnh
Khi thực hiện lệnh INSERT vào bảng GiaoDich, trong phần Messages chúng ta thấy các thông báo xuất hiện xen kẽ nhau:

"Trigger bảng B đang tác động ngược lại bảng A..."

"Đã tự động cập nhật giảm số dư người dùng."

Giải thích hiện tượng:

Thông báo này minh chứng cho việc Trigger lồng nhau (Nested Triggers) đang hoạt động.

khi Insert vào bảng A, Trigger A gọi bảng B. Ngay sau đó, Trigger trên bảng B lại kích hoạt và gọi ngược lại bảng A.

Mặc dù trong hình ảnh chưa đạt đến giới hạn lỗi (Max nesting level), nhưng nó cho thấy luồng dữ liệu đang bị chạy quẩn giữa hai bảng. Nếu logic này không có điểm dừng hoặc dữ liệu lớn hơn, nó sẽ dẫn đến lỗi Recursive Trigger và làm treo tiến trình giao dịch.


Giải thích: Khi bảng A thay đổi, nó gọi Trigger cập nhật bảng B. Khi bảng B thay đổi, nó lại gọi Trigger cập nhật ngược lại bảng A. Quá trình này tạo thành một vòng lặp vô tận. SQL Server sẽ tự động ngắt sau 32 lần lặp để tránh treo hệ thống.

4.5. Nhận xét cuối cùng

Việc thiết kế Trigger cần hết sức cẩn thận để tránh hiện tượng đệ quy (Recursive).

Nếu logic nghiệp vụ quá phức tạp và cần tác động qua lại nhiều bảng, nên sử dụng Stored Procedure để thay thế nhằm kiểm soát luồng dữ liệu tốt hơn và dễ bảo trì hơn.

Phần 5: Cursor và Duyệt dữ liệu (Kiến thức 11)
5.1. Sử dụng CURSOR để xử lý dữ liệu

Logic đặt ra: Duyệt qua danh sách các gói nạp trong bảng GoiNap. Nếu gói nạp có giá tiền lớn hơn 100,000 thì in ra thông báo phân loại là "Gói Cao Cấp", ngược lại in ra "Gói Phổ Thông".

Mã SQL sử dụng Cursor:

    DECLARE @TenGoi NVARCHAR(100);
    DECLARE @Gia MONEY;

-- 1. Khai báo Cursor
    DECLARE cur_GoiNap CURSOR FOR 
    SELECT TenGoi, GiaTien FROM [GoiNap];

-- 2. Mở Cursor
    OPEN cur_GoiNap;

-- 3. Duyệt qua từng bản ghi
    FETCH NEXT FROM cur_GoiNap INTO @TenGoi, @Gia;

    WHILE @@FETCH_STATUS = 0
    BEGIN
    IF @Gia > 100000
        PRINT N'Gói: ' + @TenGoi + N' - Phân loại: Gói Cao Cấp';
    ELSE
        PRINT N'Gói: ' + @TenGoi + N' - Phân loại: Gói Phổ Thông';

    FETCH NEXT FROM cur_GoiNap INTO @TenGoi, @Gia;
    END;

-- 4. Đóng và Giải phóng Cursor
    CLOSE cur_GoiNap;
    DEALLOCATE cur_GoiNap;

<img width="960" height="540" alt="{0729C999-F507-424E-9F74-7FDAFA2AAD9D}" src="https://github.com/user-attachments/assets/ca24d259-9997-4fb8-92f1-83d7bec5c017" />


5.2. Giải quyết bài toán không dùng CURSOR (Set-based)

Trong SQL Server, chúng ta có thể dùng lệnh SELECT kết hợp với biểu thức CASE WHEN để giải quyết bài toán trên một cách tối ưu hơn.

    Mã SQL:


    SELECT TenGoi, GiaTien,
       CASE 
           WHEN GiaTien > 100000 THEN N'Gói Cao Cấp'
           ELSE N'Gói Phổ Thông'
       END AS PhanLoai
    FROM [GoiNap];

So sánh tốc độ và hiệu năng:

Thời gian xử lý: Khi thực hiện trên tập dữ liệu lớn, cách dùng SELECT (Set-based) luôn nhanh hơn nhiều so với CURSOR.

 Nguyên nhân: Cursor phải xử lý tuần tự từng dòng (Row-by-row), tốn tài nguyên bộ nhớ. Trong khi đó, SQL Server được tối ưu để xử lý theo tập hợp dữ liệu (Set-based), giúp thực thi lệnh gần như tức thì.

<img width="960" height="540" alt="{BF099A95-E5A4-4DF7-95F5-6CFCE5F2DF54}" src="https://github.com/user-attachments/assets/2e06c96b-ebf2-4101-b6e1-1cb2092de2f7" />


5.3. Bài toán chỉ CURSOR mới giải quyết hiệu quả

Theo logic suy nghĩ của em, có những trường hợp đặc biệt mà SQL thuần rất khó giải quyết hiệu quả bằng lệnh SELECT, ví dụ: Gửi thông báo cá nhân hóa cho từng khách hàng qua hệ thống bên ngoài.

 Lý do: Khi cần gọi một Store Procedure hệ thống (như gửi Email qua sp_send_dbmail) hoặc thực thi các câu lệnh từ thư viện bên ngoài cho từng dòng dữ liệu riêng biệt, chúng ta bắt buộc phải dùng CURSOR. Việc xử lý theo tập hợp (Set-based) của SQL thuần không hỗ trợ thực hiện các lệnh đơn lẻ mang tính tuần tự này cho từng khách hàng một cách độc lập được.
