# krsmApp-ver1.03
## 解説
### @main
```modelContainer```で定義している```Reservation```と```BusinessSettings```のデータ設計をSwiftDataで保存、取得できるようにする。

```SelectView()```で```struct SelectView```を開く。

起動時に開くページを```WindowGroup```内に書く。
```
import SwiftUI
import SwiftData
@main
struct KrsmaApp:App{
    var body:some Scene{
        WindowGroup{
            SelectView()
        }
        .modelContainer(for:[Reservation.self,BusinessSettings.self])
    }
}
```
### データ設計
```@Model```で```class```を定義する。それによりデータ設計をしている。
```
//データ設計
//予約情報
@Model
class Reservation{
    var dateTime:Date
    var name:String
    var service:String
    init(dateTime:Date,name:String,service:String){
        self.dateTime=dateTime
        self.name=name
        self.service=service
    }
}
//管理者情報
@Model
class BusinessSettings{
    var holiday:Int
    var startTime:Int
    var endTime:Int
    init(holiday:Int,startTime:Int,endTime:Int){
        self.holiday=holiday
        self.startTime=startTime
        self.endTime=endTime
    }
}
```
### 利用者選択画面
```NavigationStack```で変容するための土台を作る。そして```NavigationLink```を土台の上に作る。

お客様として利用するボタンを押すと```ContentView```に移動する。

店舗管理者としてログインするボタンを押すと```MusterPassView```に移動する。
```
//利用者選択画面
struct SelectView:View{
    var body:some View{
        NavigationStack{
            Text("ヘアーサロンキリシマ")
                .font(.title)
            NavigationLink("お客様として利用"){
                ContentView()
            }
            .padding()
            NavigationLink("店舗管理者としてログイン"){
                MusterPassView()
            }
            .padding()
        }
    }
}
```
### 管理者ログイン画面
```private let musterPass="Ykousuke0715"```の部分は```private```は

「```MusterPassView```の中だけで使う。」という意味。

```let```は「一度決めた値は変更できない。」という意味。

```@State```は「SwiftUIが追跡して変わったら画面更新に反映させる。」という意味。

つまり```@State private var pass=""```では「passが変わったら今の画面だけで使えて反映する」という意味。

```SecureField```では「入力した文字を隠す」という意味。

```
Button("ログイン") {
                if pass == musterPassword {
                    loginSuccess = true
                }else{
                    loginSuccess = false
                }
```
```pass```と```musterPassword```が同じなのであればログインを管理している```loginSuccess```が```true```になりアクセス可能になる。

```.navigationDestination(isPresented: $loginSuccess)```この部分の```navigationDestination```は条件を達成したら実行されます。その条件が```isPresented:$loginSuccess```になります。

```ispresented:$loginSuccess```は```loginSuccess```が```true```であれば動くということです。
```
//管理者ログイン画面
struct MusterPassView: View {
    private let musterPassword = "Ykousuke0715"
    @State private var pass = ""
    @State private var loginSuccess = false
    var body: some View {
        VStack {
            Text("店舗管理者ログイン")
            SecureField("パスワードを入力", text: $pass)
                .textFieldStyle(.roundedBorder)
            Button("ログイン") {
                if pass == musterPassword {
                    loginSuccess = true
                }else{
                    loginSuccess = false
                }
            }
        }
        .padding()
        .navigationDestination(isPresented: $loginSuccess) {
            MusterView()
        }
    }
}
```
### 管理者編集画面
```@Query```は「SwiftDataに保存されているデータを取り出して、そのViewで使えるようにする。」という意味。

例えば```@Query private var records:[Reservation]```では、```@Query```でSwiftDataから```Reservation```を取ってきてそれを```private```でこの画面内で使える形にして```var```で情報を変更できるようにしている。それを```records```という配列に```Reservation```というデータを入れる。

```@Environment```はSwiftUIの環境に用意されている値や機能を現在のViewで使えるようにする。```modelContext```はデータを削除や追加ができる道具。

```
 if let setting = settings.first {
                SettingsEditView(setting: setting)
            } else {
                Text("設定を作成しています")
            }
```
```settings.first```は```settings```に初めの一件分のデータを指します。

```if let setting=settings.first{```は```settings```に一件分のデータがあるならばそれを```setting```として使用するということです。そしてデータが存在すれば```SettingsEditView(setting:setting)```として```SettingsEditView```に送られます。
```else```であれば```MusterView```の内容を実行する。

```
DatePicker(
                "予約者日付検索",
                selection: $searchDate,
                displayedComponents: [.date]
            )
            .datePickerStyle(.graphical)
```
```DatePicker```カレンダーから日付データを取得します。```selection:$searchDate```でどこに選択した日付を入れるかを決めて```displayedComponents:[.date]```でカレンダーから日付だけを取得する。```.datePickerStyle(.graphical)```これでページを開いている間は永続的にカレンダーを表示する。
```
Button("検索") {
                showResult = true
            }
            if showResult {
                Text("検索結果")
                List {
                    ForEach(records) { reservation in
                        if Calendar.current.isDate(
                            reservation.dateTime,
                            inSameDayAs: searchDate
                        ) {

                            VStack(alignment: .leading) {
                                Text(
                                    reservation.dateTime,
                                    format: .dateTime
                                        .hour()
                                        .minute()
                                )
                                Text("名前：\(reservation.name)")
                                Text("メニュー：\(reservation.service)")
                            }
                        }
                    }
                }
            }
```
ボタンを押すと```showResult```が```true```になり、それにより検索内容が出力される。
```List{```は一覧表示するための箱で```ForEach(records){reservation in```で```python```でいうところのfor文と同じでPythonだと

```for reservation in records```になる。

```
if Calendar.current.isDate(
    reservation.dateTime,
    inSameDayAs: searchDate
)
```

```Calendar.current```は日付の比較や計算をできるようにする。そして```.isDate(```で比較する。```reservation.dateTime```は予約データで```searchDate```は検索データです。その二つを```inSameDayAs:```で同じ日かどうかで比較していく。

日付が同じであれば
```
VStack(alignment: .leading) {
                                Text(
                                    reservation.dateTime,
                                    format: .dateTime
                                        .hour()
                                        .minute()
                                )
                                Text("名前：\(reservation.name)")
                                Text("メニュー：\(reservation.service)")
```
を実行する。
```VStack(alignment:.leading)```は```VStack```の中に入っている```VStack```を横方向にどこへそろえるかを指定している。```leading```で左揃えにしている。

```
.task {
            if settings.isEmpty {
                let setting = BusinessSettings(
                    holiday: 2,
                    startTime: 9,
                    endTime: 18
                )
                modelContext.insert(setting)
            }
        }
    }
}
```
```.task{```はViewが表示されてから実行する処理。

```if settings.isEmpty{```は```settings```が空だと実行するという意味。そのあとに```
let setting=BusinessSettings(```をsettingに入れてそれを```modelContext.insert(setting)```で挿入する。

```
//管理者編集画面
struct MusterView: View {
    @Query private var records: [Reservation]
    @Query private var settings: [BusinessSettings]
    @Environment(\.modelContext) private var modelContext
    @State private var searchDate = Date()
    @State private var showResult = false
    var body: some View {
        VStack {
            if let setting = settings.first {
                SettingsEditView(setting: setting)
            } else {
                Text("設定を作成しています")
            }
            DatePicker(
                "予約者日付検索",
                selection: $searchDate,
                displayedComponents: [.date]
            )
            .datePickerStyle(.graphical)
            Button("検索") {
                showResult = true
            }
            if showResult {
                Text("検索結果")
                List {
                    ForEach(records) { reservation in
                        if Calendar.current.isDate(
                            reservation.dateTime,
                            inSameDayAs: searchDate
                        ) {

                            VStack(alignment: .leading) {
                                Text(
                                    reservation.dateTime,
                                    format: .dateTime
                                        .hour()
                                        .minute()
                                )
                                Text("名前：\(reservation.name)")
                                Text("メニュー：\(reservation.service)")
                            }
                        }
                    }
                }
            }
        }
        .task {
            if settings.isEmpty {
                let setting = BusinessSettings(
                    holiday: 2,
                    startTime: 9,
                    endTime: 18
                )
                modelContext.insert(setting)
            }
        }
    }
}
```
### 店舗設定
```@Bindable var setting:BusinessSettings```は「SwiftDateからとってきた```BusinessSettings```の中身を、この画面から編集できるようにする。」という意味。
```
TextField(
                "休日設定",
                value: $setting.holiday,
                format: .number
            )
```
```value:$setting.holiday,```はSwiftDateの中の```BusinessSettings```の中の```holiday```の値をデータ型として```format:.number```で定義する。
```
//店舗設定
struct SettingsEditView: View {
    @Bindable var setting: BusinessSettings
    var body: some View {
        VStack {
            Text("曜日の入力方法")
            Text("1=日, 2=月, 3=火, 4=水, 5=木, 6=金, 7=土")
            TextField(
                "休日設定",
                value: $setting.holiday,
                format: .number
            )
            .textFieldStyle(.roundedBorder)
            TextField(
                "始業時間",
                value: $setting.startTime,
                format: .number
            )
            .textFieldStyle(.roundedBorder)
            TextField(
                "終業時間",
                value: $setting.endTime,
                format: .number
            )
            .textFieldStyle(.roundedBorder)
        }
    }
}
```
### 利用者予約日時画面  
```@Query private var records:[Reservation]```ではSwiftDateから```Reservation```を画面内で使えるようにしている。

```@State```で```selectedDate```で予約日時を選択する。それをデータ型にする。

```
var isBooked:Bool{
        records.contains{reservation in
            Calendar.current.isDate(
                reservation.dateTime,
                equalTo:selectedDate,
                toGranularity:.minute
            )
        }
```
```var isBooked:Bool{```は```isBooked```という名前でデータ型としては```Bool```なのでtrueかfalseで返す。

```records.contains{reservation in```の```records```はSwiftDateから持ってきた配列そして```.contains```は配列の中に条件が合うものが一件でもあるかを確かめるもの。そこでtrue,falseを選択する。

```Calendar.current.isDate```はデータを日付計算できる形にして```.isDate```で比較するようにする。

for文を回しているので```reservation.dateTime```を```selectedDate```と比較するそれを```toGranularity:.minute```により分単位で同じものかどうかを見る。
```
 DatePicker(
                "日付を選択",
                selection:$selectedDate,
                in:Date()...,
                displayedComponents:[.date]
            )
            .datePickerStyle(.graphical)
```
```DatePicker```でカレンダーを表示して日付選択する。```selection:$selectedDate,```で選択したものを```selectedDate```に情報を入れる。```in:Date()...,```で選択できる範囲を決める。```displayedComponents:[.date]```で日付を取り出すようにする。
```.datePickerStyle(.graphical)```で永続的にカレンダーを表示できるようにする。
```
 NavigationLink("時間選択"){
                DetailHourMinView(selectedDate:$selectedDate)
            }
```
```NavigationLink```で作った時間選択ボタンを押すと```DetailHourMinView```に飛んで日付データを渡す。
```
if isBooked{
                Text("この日時は予約済みです。")
                    .foregroundStyle(.red)
            }else{
                Text("この日時は予約できます。")
                    .foregroundStyle(.green)
            }
```
```isBooked```で一致するならtrueになり「予約済みです」と表示される。falseなら「予約できます」と表示させる。

```
NavigationLink(){
                CutDetailSelectView(selectedDate:selectedDate)
            }label: {
                    Text("この日時で予約")
                }
                .disabled(isBooked)
                Divider()
```
```.disabled(isBooked)```で```isBooked```がtrueならボタンを押せなくなる。ボタンを押したら```CutDetailSelectView```に飛んで```selectedDate```を渡す。
```
//利用者予約日時画面
struct ContentView:View{
    @Query private var records:[Reservation]
    @State private var selectedDate=Date()
    var isBooked:Bool{
        records.contains{reservation in
            Calendar.current.isDate(
                reservation.dateTime,
                equalTo:selectedDate,
                toGranularity:.minute
            )
        }
    }
    var body:some View{
        NavigationStack{
            //日時選択
            DatePicker(
                "日付を選択",
                selection:$selectedDate,
                in:Date()...,
                displayedComponents:[.date]
            )
            .datePickerStyle(.graphical)
            NavigationLink("時間選択"){
                DetailHourMinView(selectedDate:$selectedDate)
            }
            //予約状況
            if isBooked{
                Text("この日時は予約済みです。")
                    .foregroundStyle(.red)
            }else{
                Text("この日時は予約できます。")
                    .foregroundStyle(.green)
            }
            NavigationLink(){
                CutDetailSelectView(selectedDate:selectedDate)
            }label: {
                    Text("この日時で予約")
                }
                .disabled(isBooked)
                Divider()
        }
        .navigationTitle("予約")
        .padding()
    }
}
```
### 時間予約
```@Binding var selectedDate:Date```親画面の内容を子画面でも超苦節変更できるような形で受け取っている。
```@Environment(\.dismiss) private var dismiss```でSwiftから道具としてdismissを持ってきてdismissはページを閉じるということができる。
```
var setting:BusinessSettings?{
        settings.first
    }
```
```setting```に```BusinessSettings```の1件を入れる。
```
var endTime:Date{
        let calendar=Calendar.current
        let weekday=calendar.component(
            .weekday,
            from:selectedDate
        )
```
```endTime```にDate型で返す。

```let calendar=Calendar.current```で```calendar```という名前でカレンダー機能を使えるようにする。
```
let weekday=calendar.component(
            .weekday,
            from:selectedDate
        )
```
では```「weekday```に```selectedDate```の曜日情報を入れる。」という意味。

```
var hour=setting?.endTime ?? 18
        if weekday == 1 || weekday == 7 {
            hour=17
        }
        return calendar.date(
            bySettingHour:hour,
            minute:0,
            second:0,
            of:selectedDate
        )!
```
```var hour=setting?.endTime ?? 18```の店舗設定のsettingがあればendTimeを使うていう意味。もしなければ18時間とする。
```if weekday ==1 || weekday == 7{```だったらhour=17となる土日のみを5時までに設定する。その設定した時間が```endTime```とする。
```
return calendar.date(
            bySettingHour:hour,
            minute:0,
            second:0,
            of:selectedDate
        )!
```
```bySettinfHour:hour```で時間を終了時間のhourにしてmin,secondを０にして```of:selectedDate```でその日付の終了時間を決める。

```
var weekday:Int{
        Calendar.current.component(
            .weekday,
            from:selectedDate
        )
    }
```
```weekday```にInt型で```selectedDate```の曜日を代入。
```
var holidays:Int{
        setting?.holiday??2
    }
```
休みの設定をsettingのholidayの存在すれば代入する。
```
var startTime: Date {
    let calendar = Calendar.current
    let hour = setting?.startTime ?? 9
    return calendar.date(
        bySettingHour: hour,
        minute: 0,
        second: 0,
        of: selectedDate
    )!
```
```let calendar = Calendar.current```で```calendar```を使いカレンダー機能を使えるようにする。```let hour = setting?.startTime ?? 9```では、```setting?.startTime??9```で```setting```の```startTime```があればhourになるがなければ9になる。
```
return calendar.date(
        bySettingHour: hour,
        minute: 0,
        second: 0,
        of: selectedDate
    )!
```
hourに0分0秒を合体させたものを```selectedDate```を合体させて```startTime```に代入する。
```
var body:some View{
        if weekday == holidays{
            Text("休日")
            Button("終了"){
                dismiss()
            }
        }
        else{
            DatePicker(
                "時間を選択",
                selection:$selectedDate,
                in:startTime...endTime,
                displayedComponents:[.hourAndMinute]
            )
            Button("戻る"){
                dismiss()
            }
        }
    }
```
もし```weekday```と```holidays```が一致すれば休日となりページを終了する。
```
        else{
            DatePicker(
                "時間を選択",
                selection:$selectedDate,
                in:startTime...endTime,
                displayedComponents:[.hourAndMinute]
            )
            Button("戻る"){
                dismiss()
            }
        }
    }
```
先ほどの条件以外ならば```DatePicker```で選択範囲を表示。```in:startTime...endTime<```を範囲として扱う。
```displayedComponents:[.hourAndMinute]```で時間と分を選択させる。それにより```selectedDate```を書き換えると戻る。

```
//時間予約
struct DetailHourMinView:View{
    @Query private var settings:[BusinessSettings]
    @Binding var selectedDate:Date
    @Environment(\.dismiss) private var dismiss
    var setting:BusinessSettings?{
        settings.first
    }
    var endTime:Date{
        let calendar=Calendar.current
        let weekday=calendar.component(
            .weekday,
            from:selectedDate
        )
        var hour=setting?.endTime ?? 18
        if weekday == 1 || weekday == 7 {
            hour=17
        }
        return calendar.date(
            bySettingHour:hour,
            minute:0,
            second:0,
            of:selectedDate
        )!
    }
    var weekday:Int{
        Calendar.current.component(
            .weekday,
            from:selectedDate
        )
    }
    var holidays:Int{
        setting?.holiday??2
    }
    var startTime: Date {
    let calendar = Calendar.current
    let hour = setting?.startTime ?? 9
    return calendar.date(
        bySettingHour: hour,
        minute: 0,
        second: 0,
        of: selectedDate
    )!
    }
    var body:some View{
        if weekday == holidays{
            Text("休日")
            Button("終了"){
                dismiss()
            }
        }
        else{
            DatePicker(
                "時間を選択",
                selection:$selectedDate,
                in:startTime...endTime,
                displayedComponents:[.hourAndMinute]
            )
            Button("戻る"){
                dismiss()
            }
        }
    }
}
}
```
### サービスの選択
```
struct CutDetailSelectView: View {
    // 前の画面から受け取った日時
    let selectedDate: Date
    @Environment(\.dismiss) private var dismiss
    @Environment(\.modelContext) private var modelContext
    @State private var name = ""
    @State private var selectedService = ""
    let services: [String] = [
        "調髪：4400円",
    "調髪顔剃り無し：4100円",
    "調髪のみ：3600円",
    "アイロン：4900円",
    "SPなし：4100円",
    "顔剃りSP：3900円",
    "SPセット：2600円",
    "顔剃り：2600円",
    "セット：1200円",
    "丸刈り：3300円",
    "女性顔剃り：3300円",
    "高校生調髪：3800円",
    "高校生カット：3600円",
    "スキンフェイド：6400円",
    "中学生調髪：3300円",
    "中学生丸刈り：2100円",
    "小学生調髪：2700円",
    "小学生丸刈り：1800円",
    "乳児：3100円",
    "パーマ：9400円〜",
    "パーマ染：15000円",
    "アイパー：8100円〜",
    "Sパーマ：13000円",
    "Sパーマ前：7900円",
    "Sパーマ染：15000円",
    "Cut 白髪染：6600円",
    "Cut カラー：7600円",
    "白髪染めのみ：4200円"
    ]
    var body: some View {
        VStack(spacing: 20) {
            Text("情報入力")
                .font(.title)
                .bold()
            // 選んだ日時を表示
            Text(
                selectedDate,
                format: .dateTime
                    .year()
                    .month()
                    .day()
                    .hour()
                    .minute()
            )
            TextField("名前", text: $name)
                .textFieldStyle(.roundedBorder)
            Text("メニューを選択")
                .font(.headline)
            List {
                ForEach(services, id: \.self) { service in
                    Button {
                        selectedService = service
                    } label: {
                        HStack {
                            Text(service)
                            Spacer()
                            if selectedService == service {
                                Image(systemName: "checkmark")
                            }
                        }
                    }
                }
            }
            if selectedService != "" {
                Text("選択中：\(selectedService)")
            }
            Button("予約完了") {
                let record = Reservation(
                    dateTime: selectedDate,
                    name: name,
                    service: selectedService
                )
                modelContext.insert(record)
                dismiss()
            }
            .disabled(
                name.isEmpty ||
                selectedService.isEmpty
            )
        }
        .padding()
        .navigationTitle("予約内容")
    }
}
```

