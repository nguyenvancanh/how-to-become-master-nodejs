# Con đường trở thành master NodeJS

Để bắt đầu với một ngôn ngữ lập trình mới chưa bao giờ là điều dễ dàng với bất kỳ lập trình viên nào. Với ngôn ngữ NodeJS cũng vậy, kể cả bạn có đang là một FE develop, với nền tảng về Javascript đi chăng nữa, thì việc học NodeJS cũng có những khó khăn riêng.

Để học được NodeJs bạn cần biết cách sử dụng npm, nắm được nguyên tắc hoạt động cơ bản của Javascript. Tất cả việc này đều mất thời gian khi bạn mới bắt đầu, và đôi lúc sẽ có những khó khăn cho bạn nếu bạn không học tập một cách cẩn thận.

Trong bài viết này, tôi sẽ tổng hợp lại một vài lời khuyên cho những lập trình viên mới học NodeJS lần đầu, hy vọng với bài viết này, các bạn có thể giảm thiểu được những lỗi mắc phải khi mới học NodeJS.

# Serializing JavaScript objects

Hãy bất đầu từ điều đơn giản, nhưng lại được tìm kiếm nhiều nhất trên google: "how to serialize a JavaScript object in Node.js" (Có thể hiểu là, làm cách nào để biến chuỗi JSON thành một 'thứ gì đó' để có thể truyền đi giữa các process khác nhau).

Về cơ bản, việc serializing là biến một entity thành thứ bạn có thể tranfer được. Việc này chủ yếu áp dụng cho các object, vì các object thường rất khó khăn khi muốn transfer giữa các service, ví dụ những object có những property đặc biệt như là method, hành vi được kế thừa, hay liên kết đến các đối tượng phức tạp khác.

May mắn thay, chúng ta có JSON elements, cái mà có thể khắc phục được hầu hết các hạn chế ở trên bởi vì JSON là một object đặc biệt: 

- Bạn không thể liên kết các đối tượng JSON với nhau
- JSON được thiết kế với mục đích truyền dữ liệu
- Property của JSON có thể là bất kỳ giá trị nào, ngoại trừ function

Bây giờ hãy xem những options nào chúng ta có thể serialization trong NodeJS

## Using JSON.stringify to serialize your objects

NodeJS cho phép chúng ta access vào JSON object, với điều này bạn có thể  parse và serialize mọi chuôi JSON mà bạn cần. 

Về bản chất, stringify sẽ chuyển đổi objects của bạn về kiểu string của nó

Tuy nhiên, có một lưu ý là stringify sẽ bỏ qua một số thuộc tính vì bạn đang cố gắng chuyển đổi một object phức tạp sang định dạng ngôn ngữ bất khả thi. Stringify sẽ ignore 

- Properties với giá trị không xác định
- Properties với giá trị là một function

Bạn hay xem ví dụ duới đây

```
let testObj = {
  name: "CanhNV",
  age: 28,
  speak: function() {
    console.log("Hello world!")
  },
  address: undefined
}
let serializedObj = JSON.stringify(testObj)
testObj.speak()
console.log(serializedObj)
console.log(typeof serializedObj)
```

Khi chạy đoạn code sau, output thu được là: 

```
Hello world!
{“name”:”CanhNV”,”age”:28}
string
```
Nhữ đã nói ở trên, thuộc tính address và speak đã bị bỏ qua khi bạn gọi serializedObj, bạn có thể thấy được rõ ràng với 2 dòng console.log ở trên.

## toJSON method of complex objects

Nếu bạn code C# hay Java, những ngôn ngữ có tính hướng đối tượng hơn thì kể từ bây giờ bạn nên làm quen dần với việc sẽ không còn method toString. Trong những ngôn ngữ này, bạn sẽ gọi method này bất cứ khi nào bất cứ khi nào bạn cố gắng serialize một object và cho phép bạn custom string thu được từ việc trên.

Trong Javascript, khi bạn dùng stringify method, bạn sẽ phải định nghĩa method toJSON để custome string bạn thu được. Lưu ý rằng, khi bạn định nghĩa method thì cuối method bạn cần phải có return. Hãy xem ví dụ sau đây:

```
let testObj = {
  name: "CanhNV",
  age: 28,
  speak: function() {
    console.log("Hello world!")
  },
  toJSON: function() {
    console.log("toJSON called")
  },
  address: undefined
}
let testObj2 = {
  name: "CanhNV",
  age: 28,
  speak: function() {
    console.log("Hello world!")
  },
  toJSON: function() {
    console.log("toJSON called")
    return '{ "name": "' + this.name + '", "age": ' + this.age + ' }'
  },
  address: undefined
}
let serializedObj = JSON.stringify(testObj)
let serializedObj2 = JSON.stringify(testObj2)
testObj.speak()
console.log(serializedObj)
console.log(typeof serializedObj)
console.log(" - - - - - - - - - - - - - ")
console.log(serializedObj2)
console.log(typeof serializedObj2)
```
Khi bạn chạy code trên, kết quả thu được là:

```
toJSON called
toJSON called
Hello world!
undefined
undefined
— — — — — — — — — — — — —
“{ ”name”: ”CanhNV”, ”age”: 28 }”
string
```
Hãy nhìn vào kết quả đầu ra, khi stringify obj 1, kết quả cho chúng ta 2 line undefined. (thuộc tính toJSON là một function và không được return và kết quả của method stringify sẽ trả về type là undefined). Tuy nhiên, khi stringify object thứ 2 thì kết quả thu được về là string, nguyên nhân là hàm toJSON có return về giá trị. 

## Advanced modules (in case you need extra juice)


