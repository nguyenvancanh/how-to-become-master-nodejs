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

Phương thức stringify có thể đáp ứng được tương đối nhu cầu serialization JSON thông thường, những trong một số trường hợp đặc biệt, 2 bài toán mà method này không đáp ứng được như sau:

- Muốn serialize những phương thức an toàn để bạn có thể hủy serialize chúng và sử dụng chúng tại đích đến. 
- Vấn đề nữa là bạn xử lý rất nhiều dữ liệu bên trong JSON của bạn. (Những chuỗi JSON có kích thước lên tới hàng Gb)

Bạn có thể có các cách xử lý cho 2 bài toán này, hoặc bạn có thể xử lý logic vào bài toán của bạn hoặc tìm một module phù hợp cho nó. Một trong số package bạn bạn có thể xem xét để sử dụng là node-serialize. Tuy nhiên, bận nên lưu ý tới việc bảo mật vì sử dụng module này để gửi mã tới process đích là một rủi ro bảo mật rất lớn. 

Cách sử dụng như sau: 

```
const serialize = require("node-serialize")
var obj = {
  name: 'Bob',
  say: function() {
    return 'hi ' + this.name; 
  }
};

var objS = serialize.serialize(obj);
console.log(typeof objS === 'string');
console.log(objS)
console.log(serialize.unserialize(objS).say() === 'hi Bob')
```
Ouput thu được như sau:

```
true
{“name”:”Bob”,”say”:”_$$ND_FUNC$$_function() {n return ‘hi ‘ + this.name;n }”}
true
```
3 dòng out put cho ta biết được 3 điều: 

- Chúng ta đã serializing một object thành một chuỗi string
- Dòng thứ 2 cho biết chúng ta cách mà module này hoạt động, về cơ bản nó translate đối tượng thành 1 string mà eval có thể truyển đổi thành các câu lệnh chính xác
- Bạn không cần làm bất cứ điều gì đặc biệt để tiến hành việc serialized

Cuối cùng, nếu bạn đang làm việc với một chuỗi JSON thực sự lớn, chuỗi JSON mà bạn không thể parse hoặc serialize với method JSON.stringify, thì bạn hãy dùng module JSONStream. Hãy xem ví dụ dưới đây:

```
var fileSystem = require( "fs" );
var JSONStream = require( "JSONStream" );
var books = [
  {name: "The Philosopher's Stone", year: 1997},
  {name: "The Chamber of Secrets", year: 1998},
  {name: "The Prisoner of Azkaban", year: 1999},
  {name: "The Goblet of Fire", year:2000},
  {name: "The Order of the Phoenix", year:2003},
  {name: "The Half-Blood Prince", year:2005},
  {name: "The Deathly Hallows", year:2007}
];

var transformStream = JSONStream.stringify();
var outputStream = fileSystem.createWriteStream( __dirname + "/hpdata.json" );

transformStream.pipe( outputStream );

books.forEach( transformStream.write );
  transformStream.end();
  outputStream.on(
    "finish",
    function handleFinish() {
      console.log( "JSONStream serialization complete!" );
    }
  );
 outputStream.on(
  "finish",
  function handleFinish() {
    var transformStream = JSONStream.parse( "*" );
    var inputStream = fileSystem.createReadStream( __dirname + "/data.json" );
    inputStream
      .pipe( transformStream )
      .on(
        "data",
        function handleRecord( data ) {
          console.log( "Record (event):" , data );
        }
        )
      .on(
        "end",
        function handleEnd() {
          console.log( "JSONStream parsing complete!" );
        }
      );
   }
);
```
output thu được khi chạy chương trình là: 

```
JSONStream serialization complete!
Record (event): { name: ‘The Philosopher’s Stone’, year: 1997 }
Record (event): { name: ‘The Chamber of Secrets’, year: 1998 }
Record (event): { name: ‘The Prisoner of Azkaban’, year: 1999 }
Record (event): { name: ‘The Goblet of Fire’, year: 2000 }
Record (event): { name: ‘The Order of the Phoenix’, year: 2003 }
Record (event): { name: ‘The Half-Blood Prince’, year: 2005 }
Record (event): { name: ‘The Deathly Hallows’, year: 2007 }
JSONStream parsing complete!
```

Tất nhiên, việc xử lý các task là tùy thuộc vào bạn, các module này chỉ đơn gian là tập hợp các công cụ của NodeJS cung cấp, bạn có thể tìm hiểu để tránh việc sử dụng thư viện của bên thứ 3 trong project của mình.

## Reading command line arguments on Node.js scripts

Để chạy NodeJS thì bạn cần thực hiện câu lệnh sau:

```
$ node yourscript.js
```

Nó thật đơn giản đúng không, đương nhiên, tập lệnh của bạn có khả năng nhận các tham số, giống như bất kỳ công cụ nào khác. Và trong NodeJS, việc đọc các tham số này khá đơn giản, bằng việc thêm dòng code:

```
process.argv.forEach( (val, index) => {
  console.log(index + ': ' + val);
});
```
Để test hãy thử chạy command này

```
$ node cliparams.js test test2 test 3
```

Output thu được như sau:

```
0: /path/to/node.js/bin/node
1: /path/to/your/script/cliparams.js
2: test
3: test2
4: test
5: 3
```

Để ý rằng, chúng ta pass 3 parametter vào câu lệnh của mình, nhưng chúng ta thu về được 5. Điều nay vì tham số đầu tiên là root mà node của bạn đang được cài đặt, và tham số thứ 2 là file js chưa đoạn code của bạn. Để bỏ qua 2 tham số đầu tiên, bạn có thể custom lại việc đọc tham số như sau:

```
let args = process.argv.slice(2);
args.forEach( (val, index) => {
  console.log(index + ': ' + val);
});
```

Output lúc này thu được sẽ là:

```
1: test
2: test2
3: test
4: 3
```

Mặc định, command sẽ nhận dấu "space" để tách biệt giữa các tham số, nếu bạn muốn truyền tham số vào có dấu space thì làm như sau:

```
$ node cliparams.js “test test2 test 3”

```

Kết quả thu được như sau:

```
0: /path/to/your/bin/node
1: /path/to/your/script/cliparams.js
2: test test2 test 3
```

## Finding the current script’s file path

Đây là một trong những thao tác nhanh chóng, nhưng vô cùng hữu ích. Thông thường, những ngông ngữ javascript sẽ cung cấp cho developer một số thao tác để biết được path của file code đang thao tác. Trong NodeJS chúng ta có 2 biến để biết được những thông tin trên, bạn có thể biết được full path bao gồm đường dẫn và tên tệp. Bạn chỉ cần chạy lệnh

```
console.log(__dirname)
console.log(__filename)
```
Lưu ý rằng, giá trị của những biến này có thể được modify bởi chính bạn

