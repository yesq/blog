# WORKING WITH JSON IN GO

翻译/笔记 [https://elliot.land/post/working-with-json-in-go](https://elliot.land/post/working-with-json-in-go)

```golang
import (
    "encoding/json"
    "fmt"
)
```

理想状态下。数据的模式和 golang 里 struct 的定义一致。 

```golang
// JSON Decode
    type Person struct {
        FirstName string
        LastName  string
    }

    func main() {
        theJson := `{"FirstName": "Bob", "LastName": "Smith"}`

        var person Person
        json.Unmarshal([]byte(theJson), &person)

        fmt.Printf("%+v\n", person)
    }
```

```golang
// JSON Encode
    person := Person{"James", "Bond"}
    theJson, _ := json.Marshal(person)
```

`struct` 的 `tags` 字符串中可以定义 `json` 的键。

```golang
type Person struct {
    FirstName string `json:"fn"`
    LastName  string `json:"ln"`
}
```

Decode 的时， 传入对象中的值就是默认值。


## 动态 JSON

`struct` 的某个字段类型为 `interface{}`

`personFlexible.Name.(string)` 可以判断类型。 switch personFlexible.Name.(type)

## 其他功能

比如说检查 JSON 语法是否正确。可以用第三方库。

"github.com/xeipuuv/gojsonschema"