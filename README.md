# BINDING ADAPTER

[BINDING ADAPTER DOCUMENT](https://developer.android.com/topic/libraries/data-binding/binding-adapters)

- The Data Binding Library cho phép chỉ định 1 method được gọi để set value, cung cấp logic binding của riêng bạn (custom logic), và chỉ định loại object được trả về bằng cách sử dụng adapter

- ví dụ, ta sử dụng Binding Adapter để cung cấp custom logic hiển thị ảnh lên ImageView từ 1 URL, nếu URL bị lỗi thì ta sẽ hiển thị 1 ảnh thay thế thể hiện việc load ảnh bị lỗi
- về thư viện xử lý ảnh ta có thể sử dụng GLIDE Library

## CÁC BƯỚC CHUẨN BỊ

- khai báo sử dụng ``dataBinding`` trong Gradle
```js
android {
    // 

    buildFeatures {
        dataBinding true
    }
}
```
- khai báo class ViewModel (MainViewModel), tìm URL của 1 Image bất kỳ trên internet làm giá trị thuộc tính cho nó
```java
public class MainViewModel {
    public String url = "http://t2.gstatic.com/licensed-image?q=tbn:ANd9GcQlO_2ts2YDGtpdafB8JDZzGVfyKlmFCn7paIJmTsKhfbev0I3O-OoMwgHJUDjSTc-KbjZge4_FgB2BUqVblVM";
}
```
- vì đây là URL hiển thị ảnh trên Internet, nên cần phải khai báo xin quyền sử dụng Internet trong Android Manifest
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <application
        <!--  -->
    </application>

</manifest>
```
- tìm một ảnh dùng để thay thế hiển thị __No Image Available__ nếu URL ảnh bị lỗi, và đặt ảnh thay thế vào __drawable__
- khai báo Data trong layout (activity_main), cùng cấp với thẻ ``data`` ta khai báo 1 ViewGroup Layout bất kỳ ví dụ LinearLayout
- trong LinearLayout ta khai báo 1 ImageView chỉ set thuộc tính chiều cao và chiều ngang cho nó
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <data>
        <variable
            name="MainViewModel"
            type="com.example.adapterdatabinding.MainViewModel" />
    </data>
        </data>
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
    </LinearLayout>
</layout>
```
- khai báo Binding thực hiện liên kết Layout và ViewModel trong class control (MainActivity)
```java
public class MainActivity extends AppCompatActivity {
    private ActivityMainBinding activityMainBinding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        
        activityMainBinding = ActivityMainBinding.inflate(getLayoutInflater());
        MainViewModel mainViewModel = new MainViewModel();
        activityMainBinding.setMainViewModel(mainViewModel);
        
        setContentView(activityMainBinding.getRoot());
    }
}
```

## THƯ VIỆN LOAD ẢNH - GLIDE

- so với __Picasso__, thì __Glide__ có những ưu điểm nổi trội hơn

### CÀI ĐẶT GLIDE CHO PROJECT

[GITHUB GLIDE](https://github.com/bumptech/glide)

- khai báo implementation Glide vào dependencies trong Gradle sau đó tiến hành Sync Now để cài đặt và đồng bộ Glide vào project
```js
dependencies {
  implementation 'com.github.bumptech.glide:glide:4.15.1'
}
```

## SỬ DỤNG BINDING ADAPTER

### CÀI ĐẶT METHOD LOAD ẢNH LÊN IMAGE VIEW SỬ DỤNG BINDING ADAPTER

- trong MainActivity cài đặt method sau (method từ tài liệu của Google)
```java
    @BindingAdapter({"imageUrl", "error"})
    public static void loadImage(ImageView view, String url, Drawable error) {
        Picasso.get().load(url).error(error).into(view);
    }
```
- trong method này ta sẽ thay thế thư viện Picasso thành Glide, ``get()`` ta sẽ đổi thành ``with(view.getContext())``

```java
    @BindingAdapter({"imageUrl", "error"})
    public static void loadImage(ImageView view, String url, Drawable error) {
        Glide.with(view.getContext()).load(url).error(error).into(view);
    }
```

### KHAI BÁO THUỘC TÍNH CHO IMAGE VIEW

- ở method ``loadImage`` ta đã cài đặt trong MainActivity đang sử dụng Binding Adapter
- sau anotation ``@BindingAdapter`` ta có 2 trường ``"imageUrl"`` và ``"error"`` (2 trường này ta có thể thay đổi tùy theo nhu cầu)
- lúc này trong layout, ở ImageView ta sẽ thêm vào 2 thuộc tính, đó chính là 2 trường đã được thiết lập ở method sử dụng Binding Adapter
    - __imageUrl__
    - __error__
- từ bây giờ ta sẽ gọi là thuộc tính __imageUrl__ và thuộc tính __error__, và 2 thuộc tính này ta sử dụng Binding Adapter để custom cho ImageView, chứ các thuộc tính android cung cấp sẵn cho ImageView không có
    - thuộc tính __imageUrl__ ta set giá trị cho nó chính là URL ảnh của MainViewModel
    - thuộc tính __error__ ta set giá trị cho nó là ảnh được đặt trong ``drawable`` dùng để hiển thị trường hợp lỗi khi load ảnh
```xml
        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:imageUrl="@{MainViewModel.url}"
            app:error="@{@drawable/no_image_available}"/>
```

> khi khai báo thuộc tính custom sử dụng Binding Adapter phải sử dụng ``app:``, vì đây không phải là thuộc tính của android cung cấp mà do ta custom trong project

- logic ở đây là khi ImageView được set những giá trị tương ứng ở 2 trường ``imageUrl`` và ``error``, thì khi sử dụng Binding Adapter, nếu có khai báo 2 trường đó sau anotation, thì giá trị đó sẽ được cung cấp cho method được gắn anotation ``@BindingAdapter``

- run chương trình và mặc dù ta thấy method ``loadImage`` không được gọi ở bất kỳ đâu, nhưng do ta sử dụng Binding Adapter, nên khi ImageView được load lên layout, nó chứa 2 thuộc tính custom khai báo ở anotation ``@BindingAdapter`` của method ``loadImage`` nên giá trị của 2 thuộc tính đó sẽ tự động được đưa vào ``loadImage`` để thực hiện logic

## KẾT LUẬN

> trong Project thực tế, ta có thể sử dụng Binding Adapter để có thêm, tùy chỉnh các trường thuộc tính tùy theo nhu cầu