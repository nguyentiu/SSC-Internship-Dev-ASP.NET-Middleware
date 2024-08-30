# SSC-Internship-Dev-ASP.NET-Middleware
# Hướng Dẫn Về Middleware Trong ASP.NET Core
# Giới Thiệu
Middleware là một thành phần cốt lõi trong kiến trúc của ASP.NET Core, giúp xử lý các yêu cầu HTTP. Khi một yêu cầu được gửi tới ứng dụng ASP.NET Core, nó sẽ đi qua một chuỗi các middleware trước khi đến controller và xử lý logic nghiệp vụ. Sau đó, phản hồi HTTP sẽ đi ngược lại qua các middleware để trả về kết quả cuối cùng cho client.

## 1. Middleware Là Gì?
Middleware là những thành phần phần mềm nhỏ được gắn vào pipeline (chuỗi xử lý) của ứng dụng ASP.NET Core để xử lý các yêu cầu và phản hồi HTTP. Mỗi middleware có thể:

1. Xử lý yêu cầu: Middleware có thể thực hiện một công việc cụ thể như xác thực, logging, hoặc nén dữ liệu trước khi chuyển tiếp yêu cầu tới middleware tiếp theo.
2. Chặn yêu cầu: Middleware có thể quyết định không chuyển tiếp yêu cầu tới middleware tiếp theo mà trả về phản hồi ngay lập tức.
3. Xử lý phản hồi: Sau khi yêu cầu được xử lý bởi controller, phản hồi đi ngược lại qua chuỗi middleware và có thể được chỉnh sửa trước khi gửi lại cho client.
## 2. Pipeline Của Middleware
Pipeline của middleware trong ASP.NET Core được thiết lập trong tệp `Program.cs` hoặc `Startup.cs`. Các middleware được gọi theo thứ tự mà chúng được đăng ký.

Ví Dụ Về Pipeline Cơ Bản Trong `Program.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();
});

app.Run();
```
Trong ví dụ này:

- `UseHttpsRedirection`: Chuyển hướng yêu cầu HTTP sang HTTPS.
- `UseStaticFiles`: Xử lý yêu cầu cho các tệp tĩnh như CSS, JavaScript, hình ảnh.
- `UseRouting`: Định tuyến yêu cầu tới controller tương ứng.
- `UseAuthorization`: Kiểm tra quyền truy cập của người dùng.
- `UseEndpoints`: Định nghĩa các điểm cuối (endpoints) cho ứng dụng.
## 3. Tạo Middleware Tùy Chỉnh
Ngoài các middleware có sẵn, bạn có thể tạo middleware tùy chỉnh để xử lý các yêu cầu đặc biệt trong ứng dụng của mình.

Ví Dụ: Tạo Middleware Tùy Chỉnh Để Logging

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;

    public RequestLoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        Console.WriteLine($"Request: {context.Request.Method} {context.Request.Path}");
        await _next(context);
        Console.WriteLine($"Response: {context.Response.StatusCode}");
    }
}
```
Đăng Ký Middleware Tùy Chỉnh Trong Pipeline

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.UseMiddleware<RequestLoggingMiddleware>(); // Đăng ký middleware tùy chỉnh
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```
Trong ví dụ trên, middleware tùy chỉnh `RequestLoggingMiddleware` sẽ ghi log thông tin về mỗi yêu cầu HTTP và phản hồi.

## 4. Middleware Ngắn Gọn Bằng `Lambda Expressions`
Ngoài việc tạo các middleware tùy chỉnh thành các lớp riêng biệt, bạn có thể tạo các middleware đơn giản bằng cách sử dụng biểu thức lambda.

Ví Dụ: Middleware Đơn Giản Bằng Lambda

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.Use(async (context, next) =>
{
    Console.WriteLine("Handling request.");
    await next.Invoke();
    Console.WriteLine("Finished handling request.");
});

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```
Trong ví dụ này, middleware đơn giản sẽ ghi log khi yêu cầu bắt đầu được xử lý và khi xử lý xong.

## 5. Middleware Và Dependency Injection
Middleware cũng có thể tận dụng Dependency Injection (DI) để sử dụng các dịch vụ đã đăng ký trong container của ASP.NET Core.

Ví Dụ: Middleware Sử Dụng Dependency Injection

```csharp
public class CustomHeaderMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<CustomHeaderMiddleware> _logger;

    public CustomHeaderMiddleware(RequestDelegate next, ILogger<CustomHeaderMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task Invoke(HttpContext context)
    {
        context.Response.Headers.Add("X-Custom-Header", "This is my custom header");
        _logger.LogInformation("Custom header added.");
        await _next(context);
    }
}

// Đăng ký middleware trong Program.cs
app.UseMiddleware<CustomHeaderMiddleware>();
```
Trong ví dụ này, `CustomHeaderMiddleware` sử dụng DI để nhận `ILogger` và thêm một header tùy chỉnh vào phản hồi HTTP.

## 6. Tổng kết
Middleware là một phần không thể thiếu của ASP.NET Core, cho phép bạn can thiệp vào quá trình xử lý yêu cầu và phản hồi HTTP theo cách linh hoạt và mạnh mẽ. Bạn có thể tận dụng các middleware có sẵn hoặc tạo middleware tùy chỉnh để xử lý các yêu cầu đặc biệt của ứng dụng. Hiểu rõ về middleware sẽ giúp bạn xây dựng các ứng dụng ASP.NET Core hiệu quả và dễ bảo trì.
