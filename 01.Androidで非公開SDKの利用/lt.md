## 非公開APIの利用方法

**本文**

* テーマ：Androidの**非公開API**
* フォーカス：

  1. やりかた（リフレクション）
  2. やりかた（スタブ）
  3. できる理由（昔）
  4. 残念：今はできない
  5. できなくなった理由（しくみ）

---

## スライド2：やりかた説明（リフレクション）

**非公開APIを“名前指定”で呼ぶ**

**本文（概要サンプル）**

```kotlin
@SuppressLint("BlockedPrivateApi")
fun getSysProp(key: String): String? = try {
    val cls = Class.forName("android.os.SystemProperties")
    val get = cls.getMethod("get", String::class.java)
    get.invoke(null, key) as String
} catch (t: Throwable) { null }
```

* ポイント：**完全修飾名＋メソッドシグネチャ**が分かれば呼べる
* 但し：**互換性・審査・端末差のリスク大**

---

## スライド3：やりかた説明（スタブ）

**コンパイルだけ通す“名前解決ダミー”**

**本文（概要サンプル）**

```java
// :hidden-stubs/src/main/java/android/os/SystemProperties.java
package android.os;
public final class SystemProperties {
  private SystemProperties() {}
  public static String get(String key) {
    throw new UnsupportedOperationException("stub");
  }
}
```

```kotlin
// app/build.gradle.kts
dependencies { compileOnly(project(":hidden-stubs")) } // APKに入れない
```

* **同FQCN／同シグネチャ**で用意
* 役割：**IDE補完・型安全・ビルド通過**（実機呼び出しは別経路）
* 鉄則：**compileOnly**でAPK混入禁止

---

## スライド4：できる理由（昔）→ 残念：今はできない

**昔（〜Android 8.1）**

* `@hide`＝**SDK(android.jar)に載せないだけ**
* 実体は端末に存在 → **リフレクションで到達可能**

**今（Android 9以降）**

* **hidden API 制限**により**実行時でブロック**（`IllegalAccessError` 等）
* **targetSdkを下げても**原則回避不可
* **Playポリシー上も非推奨／リジェクト対象**

---

## スライド5：できなくなった理由（しくみ）

**3層ガードで封鎖**

1. **SDK生成層**：`@hide` / `@SystemApi` / `@TestApi` → **metalava**が**公開SDKスタブから除外**
2. **ビルド時フラグ層**：`hiddenapi-flags`（**greylist/blacklist/max-target-○** など）を各メンバに付与 → **BOOTCLASSPATH**に埋め込み
3. **実行時（ART）強制層**：リフレクション/JNI参照時に**フラグ判定→アクセス拒否**

補足：`@UnsupportedAppUsage` は**プラットフォーム内の互換注釈**で、一般アプリの免罪符ではない。
