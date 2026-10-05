### サービスの選択

`let selectedDate:Date`で前の画面から選択された予約日時を受け取る。

`@Environment(\.dismiss) private var dismiss`で現在の画面を閉じるための`dismiss`をSwiftUIの環境から持ってくる。

`@Environment(\.modelContext) private var modelContext`でSwiftDataにデータを追加するための`modelContext`を持ってくる。

`@State private var name=""`では入力された予約者の名前を`name`に入れて、値が変更された場合SwiftUIがその変更を画面に反映する。

`@State private var selectedService=""`では選択されたサービスを`selectedService`に入れる。

`let services:[String]`では選択できるサービスを文字列の配列として作っている。

```swift
Text(
    selectedDate,
    format: .dateTime
        .year()
        .month()
        .day()
        .hour()
        .minute()
)
```

`selectedDate`には前の画面で選択された日時が入っている。`format:.dateTime`で日付を表示できる形にして`.year() .month() .day() .hour() .minute()`で年、月、日、時、分を表示する。

```swift
TextField("名前", text: $name)
```

`TextField`で予約者の名前を入力する。`text:$name`で入力された内容を`name`に入れる。

```swift
List {
    ForEach(services, id: \.self) { service in
```

`List`でサービスを一覧表示するための箱を作る。

`ForEach`で`services`の中に入っているサービスを一件ずつ取り出して`service`として使用する。

`id:\.self`ではサービスの文字列そのものを一件ごとに見分けるためのIDとして使用する。

```swift
Button {
    selectedService = service
}
```

サービスのボタンを押したら、そのサービスを`selectedService`に代入する。

例えば「調髪：4400円」を押した場合は`selectedService`に「調髪：4400円」が入る。

```swift
HStack {
    Text(service)
    Spacer()
```

`HStack`で横方向に表示する。

`Text(service)`でサービス名を表示し、`Spacer()`でサービス名と右側のチェックマークの間に空間を作る。

```swift
if selectedService == service {
    Image(systemName: "checkmark")
}
```

現在選択している`selectedService`と、今表示している`service`が同じであればチェックマークを表示する。

これによってどのサービスを選択しているのか分かるようにする。

```swift
if selectedService != "" {
    Text("選択中：\(selectedService)")
}
```

`selectedService`が空ではない場合、現在選択しているサービスを「選択中：サービス名」という形で表示する。

```swift
Button("予約完了") {
    let record = Reservation(
        dateTime: selectedDate,
        name: name,
        service: selectedService
    )
```

「予約完了」ボタンを押したら`Reservation`のデータを新しく作る。

`dateTime:selectedDate`に選択した日時、`name:name`に入力した名前、`service:selectedService`に選択したサービスを入れる。

そして作った予約データを`record`に代入する。

```swift
modelContext.insert(record)
dismiss()
```

`modelContext.insert(record)`で作った`record`をSwiftDataに追加する。

そのあと`dismiss()`で現在の予約内容画面を閉じる。

```swift
.disabled(
    name.isEmpty ||
    selectedService.isEmpty
)
```

`.disabled`で予約完了ボタンを押せるかどうかを決める。

`name.isEmpty`は名前が空かどうかを確認して、`selectedService.isEmpty`はサービスが選択されていないかを確認する。

`||`は「または」という意味なので、名前が空またはサービスが選択されていない場合は予約完了ボタンを押せないようにする。

つまりこの画面では、前の画面から予約日時を受け取り、名前とサービスを選択して、その情報から`Reservation`を作りSwiftDataに保存する。
