# krsmApp-ver1.03
# 解説
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

