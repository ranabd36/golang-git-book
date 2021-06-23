# Gorilla Schema 

Installation With a correctly configured Go tool chain:

```
go get github.com/gorilla/schema
```

GitHub Link: https://github.com/gorilla/schema

Package Doc: https://pkg.go.dev/github.com/gorilla/schema

## Example 

Here's a quick example: we parse POST form values and then decode them into a struct:

```html
<form method="POST">
	<input type="text" name="Name">
	<input type="text" name="Phone">
</form>
```

```go
// Set a Decoder instance as a package global, because it caches
// meta-data about structs, and an instance can be shared safely.
var decoder = schema.NewDecoder()

type Person struct {
    Name  string
    Phone string
}

func MyHandler(w http.ResponseWriter, r *http.Request) {
    if err := r.ParseForm(); err != nil {
        // Handle error
    }

    var person Person
    // r.PostForm is a map of our POST form values
    err = decoder.Decode(&person, r.PostForm)
    if err != nil {
        // Handle error
    }

    // Do something with person.Name or person.Phone
}
```

## Documentation

**Note:** it is a good idea to set a Decoder instance as a package global, because it caches meta-data about structs, and an instance can be shared safely:

```go
var decoder = schema.NewDecoder()
```

To define custom names for fields, use a struct tag "schema". To not populate certain fields, use a dash for the name and it will be ignored:

```go
type Person struct {
    Name  string `schema:"name,required"`  // custom name, must be supplied
    Phone string `schema:"phone"`          // custom name
    Admin bool   `schema:"-"`              // this field is never set
}
```

To fill nested structs, keys must use a dotted notation as the "path" for the field. So for example, to fill the struct Person below:

```html
<form method="POST">
	<input type="text" name="Name">
	<input type="text" name="Phone.Label">
	<input type="text" name="Phone.Number">
</form>
```

```go
type Phone struct {
	Label  string
	Number string
}

type Person struct {
	Name  string
	Phone Phone
}
```

Single values are filled using the first value for a key from the source map. Slices are filled using all values for a key from the source map. So to fill a Person with multiple Phone values, like:

```html
<form method="POST">
	<input type="text" name="Name">
	<input type="text" name="Phones.0.Label">
	<input type="text" name="Phones.0.Number">
	<input type="text" name="Phones.1.Label">
	<input type="text" name="Phones.1.Number">
	<input type="text" name="Phones.2.Label">
	<input type="text" name="Phones.2.Number">
</form>
```

```go
type Person struct {
	Name   string
	Phones []Phone
}
```

**The supported field types in the struct are:**

* bool
* float variants (float32, float64)
* int variants (int, int8, int16, int32, int64)
* string
* uint variants (uint, uint8, uint16, uint32, uint64)
* struct
* a pointer to one of the above types
* a slice or a pointer to a slice of one of the above types

Unsupported types are simply ignored, however custom types can be registered to be converted.

**Custom Type Register Example:**

```go
decoder := schema.NewDecoder()
decoder.RegisterConverter(time.Time{}, timeConverter)

func timeConverter(value string) reflect.Value {
    if v, err := time.Parse("02/01/2006", value); err == nil {
        return reflect.ValueOf(v)
    }
    return reflect.Value{}
}
```

