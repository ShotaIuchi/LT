# 非公開APIの利用方法

🎺ぱふぱふ🎺

---

# はじめに

みなさんLT資料を作り込んでて、

LT発表のハードルが上がっているため、

ハードルを下げることを使命に、

"あえて"

簡素にしています。

---

# 目次

1. やりかた
    - つまんないやりかた
    - かっこいいやりかた
2. 仕組み
    - Javaの仕組み
    - Androidの仕組み

---

# やりかた（つまんない）

1. リフレクションで呼ぶ

```kotlin
fun getSysProp(key: String): String? = try {
    val cls = Class.forName("android.os.SystemProperties")  // 非公開APIのクラスオブジェクト取得
    val get = cls.getMethod("get", String::class.java)      // 非公開API取得
    get.invoke(null, key) as String                         // 非公開API実行
} catch (t: Throwable) { null }
```

→ ださい

---

# やりかた（かっこいい）

1. コンパイルだけ通す“名前解決ダミー”を用意する

```java
// :hidden-stubs/src/main/java/android/os/SystemProperties.java
package android.os;
public final class SystemProperties {
  public static String get(String key) { // 非公開API
    throw new UnsupportedOperationException("stub");  // 中身は適当
  }
}

// app/build.gradle.kts
dependencies { compileOnly(project(":hidden-stubs")) } // APKに含めない
```

2. "名前解決ダミー”で非公開SDKを呼び出す

```kotlin
// :app/src/main/java/android/iuchi/hoge.kt
fun hoge(sp: SystemProperties) : String = sp.get("hoge")
```

→ かっこいい

---

# しくみ（Java編）

Javaのライブラリ呼び出しの仕組み

※AndroidアプリはJavaで書かれている

1. ライブラリのバイトコードに“名前・型情報”が入っている
2. コンパイル時に「名前」で型解決/検証
3. 実行時に「名前 → メタデータ → 実体」へ解決して呼び出し

→ 今回は2.部分を騙してる

---

# しくみ（Android編）

* 非公開SDKの定義

```java
// frameworks/base/core/java/android/view/Window.java
    /** @hide */
    @UnsupportedAppUsage
    public void setCloseOnTouchOutside(boolean close) {
        mCloseOnTouchOutside = close;
        mSetCloseOnTouchOutside = true;
    }
```

→ ビルドすると

---

# しくみ（Android編）（ビルドすると）

* 端末用（組み込みOS）のバイナリ（framework.jarなど）

```java
public void setCloseOnTouchOutside(boolean close)
```
が含まれる

* SDK用のバイナリ（android.jarなど）

```java
// public void setCloseOnTouchOutside(boolean close)
```
が無い！

→ すなわち・・・？？？

---

# しくみ（Android編）

非公開SDKは

- SDK用のバイナリに入っていないだけで、
- 実際の実行用バイナリには入っているので

名前を解決さえすれば、非公開SDKだって呼び放題！！！

---

# でも・・・

今のAndroidではできない

→ 理由は？

---

# でも・・・

今のAndroidではできない

↓ 理由

長くなるからまたいつか
