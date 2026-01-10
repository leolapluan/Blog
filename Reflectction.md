# Java Reflection — Bảng kiến thức liên quan

> Ghi chú: Bạn có thể đang muốn nói “reflection” (hay bị gõ nhầm thành “reflexion”).

## Bảng kiến thức Reflection (Java)

| Mảng kiến thức | API / Keyword chính | Dùng để làm gì | Ví dụ nhanh (ý tưởng) | Lưu ý / liên kết kiến thức |
|---|---|---|---|---|
| Điểm vào Reflection | `Class<?>`, `obj.getClass()` | Lấy “metadata” của class từ object | `user.getClass()` | Gắn với **ClassLoader**, **Module system** |
| Load class theo tên | `Class.forName("pkg.A")` | Nạp class runtime theo string | Plugin / JDBC driver | Dễ gặp `ClassNotFoundException` |
| Tạo object động | `clazz.getDeclaredConstructor().newInstance()` | Khởi tạo khi không biết class lúc compile | Factory/DI container | Ném nhiều exception; ưu tiên constructor rõ ràng khi có thể |
| Duyệt constructor | `getConstructors()`, `getDeclaredConstructors()` | Xem các constructor public / tất cả | Chọn đúng constructor để `newInstance` | `declared` gồm cả private |
| Duyệt method | `getMethods()`, `getDeclaredMethods()` | Liệt kê method public / tất cả | Build router, test framework | Overload → phải match đúng signature |
| Gọi method runtime | `Method.invoke(obj, args...)` | Gọi method khi biết tên lúc runtime | `invoke("setName", "A")` | Chậm hơn gọi trực tiếp; dễ lỗi kiểu tham số |
| Duyệt field | `getFields()`, `getDeclaredFields()` | Truy cập data field runtime | Mapper object ↔ JSON/DB | Field private cần mở access |
| Đọc/ghi field | `Field.get(obj)`, `Field.set(obj, val)` | Bypass getter/setter (khi cần) | Serializer/deserializer | Cẩn thận encapsulation & immutability |
| Bỏ qua access control | `setAccessible(true)` | Truy cập private/protected | Framework nội bộ | JDK mới + module có thể bị hạn chế; rủi ro bảo mật |
| Annotation runtime | `@interface`, `isAnnotationPresent`, `getAnnotation` | Đọc metadata cấu hình | Spring, JUnit, Validation | Liên kết mạnh với **AOP**, **DI**, **Bean scanning** |
| Generic type info | `getGenericType()`, `ParameterizedType` | Đọc kiểu generic (hạn chế) | `List<String>` mapping | **Type erasure**: runtime không luôn đủ thông tin |
| Proxy động (JDK) | `Proxy.newProxyInstance`, `InvocationHandler` | Tạo object “bọc” interface | Logging, retry, auth | Chỉ dùng cho **interface**; class proxy → thường dùng Bytecode libs |
| Caching metadata | Cache `Method/Field` (Map) | Tối ưu performance khi reflect nhiều | Cache theo `Class` | Reflection tốn chi phí; cache là best practice |
| So sánh với MethodHandle | `MethodHandles.lookup()` | Gọi động nhanh hơn reflection | High-performance invoke | Liên quan **invokedynamic**, JVM internals |
| Exception thường gặp | `IllegalAccessException`, `InvocationTargetException` | Bắt lỗi khi reflect/invoke | Unwrap nguyên nhân thật | `InvocationTargetException.getCause()` là “lỗi gốc” |
| Use cases chuẩn | DI/IoC, ORM, JSON mapper, test runner | Tự động wiring & mapping | Spring/Hibernate/Jackson | Liên kết: **Annotations + Classpath scanning + Proxies** |

---

## Mini mindmap liên kết kiến thức (để học nhanh)

- Reflection ↔ **Annotations** (metadata) ↔ **Framework DI/AOP** (Spring)
- Reflection ↔ **Dynamic Proxy / AOP** (wrap method calls)
- Reflection ↔ **ClassLoader** (nạp class/plugin)
- Reflection ↔ **Generics/Type erasure** (vì sao runtime thiếu kiểu)
- Reflection ↔ **Security/Modules** (vì sao `setAccessible` đôi khi bị chặn)
- Reflection ↔ **Performance** (cache / MethodHandles)

---

## Checklist khi dùng Reflection (đỡ “dính bug”)

- Ưu tiên `getDeclaredXxx()` khi cần private; nhớ `setAccessible(true)` (và chấp nhận rủi ro).
- Khi `invoke`: match đúng **parameter types** (overload dễ nhầm).
- Luôn cache `Method/Field/Constructor` nếu gọi lặp nhiều.
- Khi lỗi invoke: kiểm tra `InvocationTargetException.getCause()`.
