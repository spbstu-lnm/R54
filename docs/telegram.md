https://github.com/TelegramMessenger/Telegram-Android (v13.2.2 on master)
___
**Tools:** apktool, JADX, cve-bin-tool (+Syft, OSV-SCALIBR, LibRARIAN, Semantic Copycat BinarySniffer & MobSF)
**DBs:** NVD, CVE, OSV, Github Security Advisories
___
- APK downloaded from https://telegram.org/android
- APK extracted via `java -jar apktool d Telegram.apk` (got some warnings?? mb a non-issue)
- `file *.so` (under WSL2 Ubuntu) executed in `Telegram/lib/arm64-v8a`
```output
lnm@lnm-pc:/mnt/s/Files/Polytech/5/reverse/repos/R54/local/telegram/Telegram/lib/arm64-v8a$ file *.so
libdatastore_shared_counter.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, for Android 21, built by NDK r25c (9519653), BuildID[sha1]=17db37bd6770ac00dd2d1d2828839fd23a7959a3, stripped
liblanguage_id_l2c_jni.so:      ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, for Android 21, built by NDK r27 (12077973), BuildID[md5/uuid]=4e1cbc7b2a12b63992ad6bcaaa139a2b, stripped
libtmessages.49.so:             ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, for Android 23, built by NDK r27c (12479018), BuildID[sha1]=5359f7ec7e5327b2ccffea5d6c47c327464ea702, stripped
libtwidget.so:                  ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, for Android 23, built by NDK r27c (12479018), BuildID[sha1]=9157ba519bdfde83d97a6f005db6765c5f33370a, stripped
```
- `readelf -h libtmessages.49.so`
```output
lnm@lnm-pc:/mnt/s/Files/Polytech/5/reverse/repos/R54/local/telegram/Telegram/lib/arm64-v8a$ readelf -h libtmessages.49.so
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          22134296 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         27
  Section header string table index: 26
```
- `readelf -d libtmessages.49.so`
```output
lnm@lnm-pc:/mnt/s/Files/Polytech/5/reverse/repos/R54/local/telegram/Telegram/lib/arm64-v8a$ readelf -d libtmessages.49.so

Dynamic section at offset 0x15087c0 contains 34 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libjnigraphics.so]
 0x0000000000000001 (NEEDED)             Shared library: [liblog.so]
 0x0000000000000001 (NEEDED)             Shared library: [libz.so]
 0x0000000000000001 (NEEDED)             Shared library: [libGLESv2.so]
 0x0000000000000001 (NEEDED)             Shared library: [libEGL.so]
 0x0000000000000001 (NEEDED)             Shared library: [libandroid.so]
 0x0000000000000001 (NEEDED)             Shared library: [libOpenSLES.so]
 0x0000000000000001 (NEEDED)             Shared library: [libm.so]
 0x0000000000000001 (NEEDED)             Shared library: [libdl.so]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so]
 0x000000000000000e (SONAME)             Library soname: [libtmessages.49.so]
 0x000000000000001e (FLAGS)              SYMBOLIC BIND_NOW
 0x000000006ffffffb (FLAGS_1)            Flags: NOW
 0x0000000000000007 (RELA)               0xd048
 0x0000000000000008 (RELASZ)             1216680 (bytes)
 0x0000000000000009 (RELAENT)            24 (bytes)
 0x000000006ffffff9 (RELACOUNT)          50644
 0x0000000000000017 (JMPREL)             0x1360f0
 0x0000000000000002 (PLTRELSZ)           9120 (bytes)
 0x0000000000000003 (PLTGOT)             0x1517c78
 0x0000000000000014 (PLTREL)             RELA
 0x0000000000000006 (SYMTAB)             0x2f8
 0x000000000000000b (SYMENT)             24 (bytes)
 0x0000000000000005 (STRTAB)             0x6130
 0x000000000000000a (STRSZ)              28435 (bytes)
 0x000000006ffffef5 (GNU_HASH)           0x5538
 0x0000000000000019 (INIT_ARRAY)         0x150c768
 0x000000000000001b (INIT_ARRAYSZ)       88 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x150c758
 0x000000000000001c (FINI_ARRAYSZ)       16 (bytes)
 0x000000006ffffff0 (VERSYM)             0x4e88
 0x000000006ffffffe (VERNEED)            0x54d4
 0x000000006fffffff (VERNEEDNUM)         3
 0x0000000000000000 (NULL)               0x0
```
___
# Attempt 2
Moved to Ubuntu 26.04 bare-metal.
- Installed Android Studio 2025.1.4, Android NDK 27.2.12479018 and Android SDK 36
- Cloned repo, downloaded APK
- Installed JDK 17
- `./gradlew :TMessagesProj:dependencies --configuration releaseRuntimeClasspath > dependencies.txt`
- ***From `dependencies.txt`***: `AndroidX DataStore 1.1.7`, `ML Kit Language Identification 17.0.6`
- `grep -E 'LIBAVUTIL_VERSION_(MAJOR|MINOR|MICRO)' TMessagesProj/jni/third_party/ffmpeg/libavutil/version.h`
```output
#define LIBAVUTIL_VERSION_MAJOR  60
#define LIBAVUTIL_VERSION_MINOR  26
#define LIBAVUTIL_VERSION_MICRO 103
<...>
```
- `grep -E 'LIBAVCODEC_VERSION_(MAJOR|MINOR|MICRO)' TMessagesProj/jni/third_party/ffmpeg/libavcodec/version.h`
```output
#define LIBAVCODEC_VERSION_MINOR  28
#define LIBAVCODEC_VERSION_MICRO 103
<...>
```
- `grep -E 'LIBAVCODEC_VERSION_(MAJOR|MINOR|MICRO)' TMessagesProj/jni/third_party/ffmpeg/libavcodec/version_major.h TMessagesProj/jni/third_party/ffmpeg/libavcodec/version.h`
```output
TMessagesProj/jni/third_party/ffmpeg/libavcodec/version_major.h:#define LIBAVCODEC_VERSION_MAJOR  62
<...>
TMessagesProj/jni/third_party/ffmpeg/libavcodec/version.h:#define LIBAVCODEC_VERSION_MINOR  28
TMessagesProj/jni/third_party/ffmpeg/libavcodec/version.h:#define LIBAVCODEC_VERSION_MICRO 103
<...>
```
- ***From git repo***: `libavutil 60.26.103`, `libavcodec 62.28.103`, `FFmpeg 8.1.3` (based on `libavutil` & `libavcodec` versions)
- `grep -nE 'SQLITE_VERSION|SQLITE_SOURCE_ID' $(find TMessagesProj/jni -name sqlite3.h -o -name sqlite3.c)`
```output
<...>
TMessagesProj/jni/sqlite/sqlite3.h:149:#define SQLITE_VERSION        "3.39.3"
TMessagesProj/jni/sqlite/sqlite3.h:150:#define SQLITE_VERSION_NUMBER 3039003
TMessagesProj/jni/sqlite/sqlite3.h:151:#define SQLITE_SOURCE_ID      "2022-09-05 11:02:23 4635f4a69c8c2a8df242b384a992aea71224e39a2ccab42d8c0b0602f1e826e8"
<...>
TMessagesProj/jni/td/sqlite/sqlite/sqlite3.c:1168:#define SQLITE_VERSION        "3.31.0"
TMessagesProj/jni/td/sqlite/sqlite/sqlite3.c:1169:#define SQLITE_VERSION_NUMBER 3031000
TMessagesProj/jni/td/sqlite/sqlite/sqlite3.c:1170:#define SQLITE_SOURCE_ID      "2020-01-22 18:38:59 f6affdd41608946fcfcea914ece149038a8b25a62bbe719ed2561c649b86alt1"
<...>
```
- ***From git repo***: `SQLite 3.39.3`, `SQLite 3.31.0` (used in `TDLib`)
- `strings -a Telegram/lib/arm64-v8a/libtmessages.49.so | grep -i 'libopus'`
```output
libopus unknown-fixed
libopusdec
libopus
```
- ***From git repo and APK***: `libopus unknown-fixed`

> [!NOTE] Отчёт
> В составе `libtmessages.49.so` обнаружены признаки использования библиотеки Opus: строковые идентификаторы `libopus unknown-fixed`, `libopusdec`, а также JNI-функции Telegram, предназначенные для обработки аудиофайлов Opus. В конфигурации сборки Telegram библиотека подключается в виде предварительно собранного статического архива `libopus.a`. Следовательно, код Opus включён непосредственно в нативную библиотеку Telegram. Идентификатор версии `unknown-fixed` не позволяет установить точный upstream-релиз компонента.

- `strings -a Telegram/lib/arm64-v8a/libtmessages.49.so | grep -E 'Lavu|Lavc|Lavf'`
```output
Lavf62.12.103
<...>
Lavc62.28.103
<...>
```
- ***From APK***: `libavcodec 62.28.103` & `libavformat 62.12.103` $\to$ `FFmpeg 8.1.3`
- `strings -a Telegram/lib/arm64-v8a/libtmessages.49.so | grep -E '3\.39\.3|2022-09-05|4635f4a69c8c2a8d'`
```output
2022-09-05 11:02:23 4635f4a69c8c2a8df242b384a992aea71224e39a2ccab42d8c0b0602f1e826e8
3.39.3
```
- ***From APK***: `SQLite 3.39.3`
- ***Selected three components for analysis***: `SQLite 3.39.3`, `FFmpeg 8.1.3` & `opus unknown-fixed`
- `git log -1 --format='%H%n%cs%n%s'`
```pager
21f7d109da6ab3506c9e9577f98c6a38771ab335
2026-11-12
update to 13.2.2 (7303)
```
- `aapt dump badging Telegram.apk | head -n 4`
```output
package: name='org.telegram.messenger.web' versionCode='73039' versionName='13.2.2' platformBuildVersionName='16' platformBuildVersionCode='36' compileSdkVersion='36' compileSdkVersionCodename='16'
install-location:'auto'
sdkVersion:'23'
targetSdkVersion:'36'
```
- `versionName='13.2.2'` matches version from git log: ***source code from git repo likely matches that of 13.2.2 APK***

> [!NOTE] Отчёт
> В составе `libtmessages.49.so` обнаружена SQLite версии 3.39.3, которая входит в диапазон версий, затронутых CVE-2025-7458. В машинном коде идентифицирована функция, соответствующая `wherePathSolver()`. Установлено, что она записывает значение `isOrdered` в поле `nOBSat`, а последующие проверки не демонстрируют ограничения этого значения количеством выражений `ORDER BY`. Полученные результаты согласуются с наличием уязвимой логики в исследуемой версии SQLite. Для подтверждения эксплуатируемости необходимо установить достижимость соответствующего сценария через интерфейсы приложения.

- ***From src***: `tflite` likely bundled with `mlkit`
```
Telegram
  └── com.google.mlkit:language-id:17.0.6
        └── liblanguage_id_l2c_jni.so
              └── TensorFlow Lite (bundled)
```
- `wget https://dl.google.com/dl/android/maven2/androidx/datastore/datastore-core-android/1.1.7/datastore-core-android-1.1.7.aar`
- `unzip -q datastore-core-android-1.1.7.aar -d aar`
- `sha256sum aar/jni/arm64-v8a/libdatastore_shared_counter.so ~/dev/polytech/5/reverse/R54/local/telegram/apk/Telegram/lib/arm64-v8a/libdatastore_shared_counter.so`
```output
d3e48717c9aa147e0ab21063ba0e8e0211cabf8bf40b222640829519edbf58e1  aar/jni/arm64-v8a/libdatastore_shared_counter.so
d3e48717c9aa147e0ab21063ba0e8e0211cabf8bf40b222640829519edbf58e1  /home/lnm/dev/polytech/5/reverse/R54/local/telegram/apk/Telegram/lib/arm64-v8a/libdatastore_shared_counter.so
```
- ***Confirmed via SHA-256 hash***: AndroidX DataStore 1.1.7
- ***Confirmed via SHA3-256 hash***: SQLite 3.39.3 (Amalgamation w/o any changes from Telegram)

___
# Telegram: Report Draft
# 1. Исследование программного обеспечения Telegram

## 1.1. Общие сведения об исследуемом программном обеспечении

В качестве объекта исследования было выбрано приложение Telegram для операционной системы Android. Для анализа использовался APK-файл версии 13.2.2 (versionCode 73039), предназначенный для устройств с Android версии не ниже 6.0 (API 23). Приложение содержит нативные библиотеки для четырёх архитектур: ARM64 (arm64-v8a), ARMv7 (armeabi-v7a), x86 и x86-64 (x86_64).

Для проведения статического анализа APK-файл был распакован, после чего исследованы содержимое каталога `lib/`, заголовки ELF-файлов, таблицы динамических зависимостей, экспортируемые символы и строковые константы. Для подробного исследования была выбрана архитектура ARM64, поскольку она позволяет последовательно анализировать библиотеки одного ABI без необходимости учитывать различия между архитектурами.

В APK обнаружены четыре уникальные нативные библиотеки:

|Библиотека|Архитектура|Размер (ARM64, приблизительно)|Назначение|
|---|---|---|---|
|`libdatastore_shared_counter.so`|AArch64|7 КБ|Реализация нативного механизма счётчиков AndroidX DataStore|
|`liblanguage_id_l2c_jni.so`|AArch64|996 КБ|Нативная реализация функциональности определения языка текста|
|`libtmessages.49.so`|AArch64|22 МБ|Основная нативная библиотека Telegram, содержащая сетевую, медиальную, графическую и другие подсистемы|
|`libtwidget.so`|AArch64|5,3 МБ|Нативная библиотека, используемая компонентами интерфейса Telegram|

Все четыре библиотеки представлены в APK для каждого из поддерживаемых ABI. Исследование ELF-заголовков показало, что они являются динамически загружаемыми разделяемыми объектами ELF. При этом наличие динамической библиотеки не означает, что все используемые ею сторонние компоненты также представлены отдельными `.so`-файлами: часть зависимостей может быть статически скомпонована непосредственно с исследуемой библиотекой.

## 1.2. Анализ нативных библиотек

Для каждой библиотеки были исследованы ELF-заголовки, секции и таблицы символов, динамические зависимости (`DT_NEEDED`), а также строковые константы. Все четыре библиотеки содержат сведения, позволяющие определить целевую архитектуру и среду выполнения. При этом значительная часть символов удалена: библиотеки собраны с использованием stripping, поэтому полноценная отладочная информация и имена многих внутренних функций отсутствуют.

Анализ динамических зависимостей показал, что библиотеки используют стандартные системные компоненты Android, в частности `libc.so`, `libm.so` и `libdl.so`. В зависимости от назначения также встречаются `liblog.so`, `libandroid.so`, `libjnigraphics.so`, `libEGL.so`, `libGLESv2.so` и другие системные библиотеки.

Особый интерес представляет `libtmessages.49.so`: в её таблице динамических зависимостей отсутствуют отдельные библиотеки FFmpeg, SQLite, Opus и WebRTC. Вместе с тем анализ экспортируемых JNI-функций, строковых констант и исходного кода Telegram подтвердил присутствие соответствующих компонентов внутри этой библиотеки. Это свидетельствует об использовании статической компоновки для ряда сторонних зависимостей.

## 1.3. Идентифицированные сторонние компоненты

В ходе исследования были обнаружены следующие сторонние программные компоненты.

|Компонент|Предполагаемая версия|Исполняемый файл|Тип включения|Достоверность идентификации|
|---|---|---|---|---|
|AndroidX DataStore|1.1.7|`libdatastore_shared_counter.so`|Нативная библиотека|Очень высокая: SHA-256 совпадает с официальной библиотекой из Google Maven|
|Google ML Kit Language Identification|17.0.6|`liblanguage_id_l2c_jni.so`|Нативная реализация SDK|Высокая: подтверждено графом Gradle-зависимостей и JNI-символами|
|TensorFlow Lite|Не установлена|`liblanguage_id_l2c_jni.so`|Встроенный нативный компонент|Высокая для факта присутствия, недостаточная для определения версии|
|FFmpeg|8.1.3|`libtmessages.49.so`|Статическая компоновка|Высокая: совпадают идентификаторы версий библиотек FFmpeg и обнаружены соответствующие JNI-функции|
|SQLite|3.39.3|`libtmessages.49.so`|Статическая компоновка|Очень высокая: исходный файл SQLite из репозитория Telegram идентичен официальной amalgamation указанной версии|
|WebRTC|Не установлена|`libtmessages.49.so`|Статическая компоновка|Высокая для факта присутствия, точная версия не установлена|
|Opus|Не установлена|`libtmessages.49.so`|Статическая компоновка|Высокая: подтверждено конфигурацией сборки и наличием соответствующих исходников|
|Opusfile|Не установлена|`libtmessages.49.so`|Компиляция исходных файлов в состав библиотеки|Высокая: подтверждено перечнем исходных файлов в CMake|

Google ML Kit Language Identification и TensorFlow Lite следует учитывать как отдельные компоненты. Версия 17.0.6 относится к ML Kit, а не к TensorFlow Lite. В бинарном файле обнаружены характерные строки, имена пространств имён и пути исходных файлов TensorFlow Lite, однако точную версию этого компонента установить не удалось. При этом ML Kit следует рассматривать как зависимость приложения и поставщика нативной функциональности, а TensorFlow Lite — как обнаруженный OSS-компонент.

### 1.3.1. AndroidX DataStore

В библиотеке `libdatastore_shared_counter.so` обнаружены JNI-функции с именами, соответствующими нативному механизму Shared Counter из AndroidX DataStore. В частности, экспортируются функции `Java_androidx_datastore_core_NativeSharedCounter_nativeCreateSharedCounter`, `nativeGetCounterValue`, `nativeIncrementAndGetCounterValue` и `nativeTruncateFile`.

Для уточнения версии была исследована конфигурация Gradle-зависимостей исходного проекта Telegram. Команда `dependencyInsight` показала использование `androidx.datastore:datastore-core-android:1.1.7`. Дополнительно была получена официальная библиотека из Google Maven, после чего выполнено сравнение SHA-256 контрольных сумм нативных файлов.

Контрольная сумма библиотеки из APK и файла, извлечённого из официального AAR-пакета AndroidX DataStore 1.1.7, совпала:

`d3e48717c9aa147e0ab21063ba0e8e0211cabf8bf40b222640829519edbf58e1`

Таким образом, установлено, что нативная библиотека из исследуемого APK побайтно совпадает с официальной реализацией AndroidX DataStore версии 1.1.7.

### 1.3.2. Google ML Kit Language Identification и TensorFlow Lite

В библиотеке `liblanguage_id_l2c_jni.so` обнаружены JNI-функции, связанные с классами Google ML Kit Language Identification. В частности, экспортируются `Java_com_google_mlkit_nl_languageid_bundled_internal_ThickLanguageIdentifier_nativeDestroy`, `nativeIdentifyPossibleLanguages` и `nativeInitFromBuffer`.

Исследование Gradle-зависимостей подтвердило использование компонента `com.google.mlkit:language-id:17.0.6`. Это позволяет установить версию ML Kit, интегрированного в приложение.

Помимо этого, в строковых константах библиотеки обнаружены характерные идентификаторы TensorFlow Lite: пути к исходным файлам из каталога `third_party/tensorflow/lite/`, сообщения об инициализации TensorFlow Lite Runtime, имена классов и структур `tflite::Subgraph`, `tflite::ArenaPlanner`, `TfLiteDelegate` и другие характерные обозначения.

Совокупность этих признаков подтверждает присутствие кода TensorFlow Lite в составе `liblanguage_id_l2c_jni.so`. Отдельная динамическая зависимость от TensorFlow Lite отсутствует. Точная версия TensorFlow Lite по имеющимся данным не установлена, поэтому версия ML Kit не должна использоваться в качестве её замены.

### 1.3.3. FFmpeg

В библиотеке `libtmessages.49.so` обнаружены идентификаторы FFmpeg:

- `Lavf62.12.103`;
    
- `Lavc62.28.103`.
    

Они соответствуют версиям libavformat 62.12.103 и libavcodec 62.28.103, входящим в состав FFmpeg 8.1.3. Дополнительное подтверждение получено при сопоставлении этих идентификаторов с официальными заголовочными файлами соответствующего выпуска FFmpeg.

Присутствие FFmpeg также подтверждается экспортируемыми JNI-функциями, используемыми для работы с медиаданными. Среди них функции инициализации и освобождения декодера, отправки пакетов, получения кадров и их обработки. В исходном коде Telegram соответствующие функции реализованы в `TMessagesProj/jni/ffmpeg_jni.cc`.

Вызовы API FFmpeg обнаружены и в других частях нативного кода Telegram. В частности, используются функции `avformat_open_input`, `avformat_find_stream_info`, `avcodec_find_decoder`, `avcodec_open2`, `avcodec_send_packet`, `avcodec_receive_frame`, а также функции библиотек libswresample и libswscale.

Отсутствие отдельных библиотек `libavcodec.so` и `libavformat.so` в списке `DT_NEEDED` при наличии соответствующих идентификаторов и функций указывает на статическую компоновку FFmpeg с основной нативной библиотекой Telegram.

### 1.3.4. SQLite

В `libtmessages.49.so` обнаружены JNI-функции, предназначенные для взаимодействия с SQLite. Они связаны с классами `SQLiteDatabase`, `SQLitePreparedStatement` и `SQLiteCursor`, используемыми Telegram для работы с локальной базой данных.

Строковые константы библиотеки содержат номер версии SQLite 3.39.3, дату `2022-09-05 11:02:23` и идентификатор исходного кода. Дополнительная проверка была выполнена на уровне исходного кода: файл `TMessagesProj/jni/sqlite/sqlite3.c` из репозитория Telegram был сопоставлен с официальной amalgamation SQLite 3.39.3. Контрольные суммы SHA3-256 совпали:

`2fc273cf8032b601c9e06207efa0ae80eb73d5a1d283eb91096c815fa9640257`

Это подтверждает, что исходный файл SQLite в репозитории Telegram идентичен официальной amalgamation версии 3.39.3. В совокупности с идентификаторами в бинарном файле и JNI-обёртками это позволяет с высокой достоверностью идентифицировать SQLite 3.39.3 как компонент `libtmessages.49.so`.

Отдельная динамическая библиотека SQLite в зависимостях ELF не обнаружена, что согласуется со статической компоновкой.

### 1.3.5. WebRTC

Присутствие WebRTC в `libtmessages.49.so` подтверждается большим количеством экспортируемых JNI-функций, связанных с классами `org.webrtc`. Среди них функции управления `PeerConnection`, `PeerConnectionFactory`, `DataChannel`, `RtpSender`, `RtpReceiver`, `RtpTransceiver` и `MediaStream`.

Дополнительное подтверждение получено при анализе исходного репозитория Telegram. В каталоге `TMessagesProj/jni/voip/webrtc/` присутствуют исходные файлы WebRTC, а конфигурация CMake содержит ссылки на соответствующие компоненты и параметры сборки, включая поддержку VP9, H.264, SCTP и Opus.

Таким образом, WebRTC используется в составе нативной реализации Telegram и скомпонован с основной библиотекой. Точная версия WebRTC по имеющимся признакам не установлена.

### 1.3.6. Opus и Opusfile

Использование аудиокодека Opus подтверждается конфигурацией сборки Telegram. В `TMessagesProj/jni/CMakeLists.txt` библиотека `libopus` объявлена как импортируемая статическая библиотека, расположенная в каталоге `prebuild/lib/${ANDROID_ABI}/libopus.a`. Скрипт `build_opus.sh` указывает на сборку статической библиотеки Opus из исходных файлов, расположенных в `third_party/xiph/opus`.

Помимо этого, в конфигурации сборки обнаружены исходные файлы Opusfile: `info.c`, `internal.c`, `opusfile.c` и `stream.c`. Они компилируются непосредственно в состав нативного проекта Telegram.

В бинарном файле присутствуют JNI-функции, связанные с обработкой Opus-файлов, включая объединение и обрезку аудиофайлов, проверку формата и получение звуковой формы. Также обнаружена интеграция Opus с FFmpeg и WebRTC.

Отдельные динамические библиотеки Opus и Opusfile в `DT_NEEDED` отсутствуют. Это согласуется с использованием статической библиотеки Opus и компиляцией исходных файлов Opusfile в состав нативной библиотеки Telegram. Точные версии обоих компонентов установить не удалось.

## 1.4. Исходный код Telegram и сопоставление версий

Для дополнительного подтверждения состава нативной библиотеки был исследован официальный исходный репозиторий Telegram. Используемая ревизия репозитория:

`21f7d109da6ab3506c9e9577f98c6a38771ab335`

Данная ревизия относится к обновлению приложения до версии 13.2.2 (7303), что согласуется с версией исследуемого APK 13.2.2. Совпадение версии приложения и ревизии исходного репозитория позволяет использовать исходный код для анализа структуры и назначения обнаруженных компонентов.

При этом само по себе совпадение версии приложения не доказывает, что APK был собран именно из указанной ревизии. Для отдельных компонентов использовались более строгие методы проверки: сопоставление идентификаторов версий, исследование JNI-функций, анализ конфигурации сборки и сравнение контрольных сумм.

Наиболее сильное подтверждение получено для AndroidX DataStore 1.1.7 и SQLite 3.39.3, для которых удалось выполнить проверку идентичности исходных или бинарных файлов. Для FFmpeg версия 8.1.3 подтверждается согласованными идентификаторами нескольких входящих в него библиотек. Для WebRTC, Opus и Opusfile точные версии не установлены.

## 1.5. Исследование уязвимости SQLite CVE-2025-7458

В качестве уязвимости для подробного исследования выбрана CVE-2025-7458, затрагивающая SQLite. Выбор обусловлен тем, что в составе `libtmessages.49.so` была идентифицирована версия SQLite 3.39.3, входящая в диапазон затронутых версий.

Согласно описанию уязвимости, причиной является целочисленное переполнение в функции `sqlite3KeyInfoFromExprList`. Уязвимость затрагивает версии SQLite начиная с 3.39.2 и заканчивая 3.41.1; исправление включено в SQLite 3.41.2.

При обработке специально сформированного SQL-запроса, содержащего большое количество выражений в конструкции `ORDER BY`, возможно переполнение целочисленного значения. В зависимости от условий эксплуатации это может привести к отказу в обслуживании или раскрытию данных из памяти процесса. Для реализации атаки необходимо иметь возможность выполнить произвольный SQL-запрос.

Исследуемая версия SQLite 3.39.3 входит в указанный диапазон затронутых версий. Следовательно, соответствующий уязвимый исходный код присутствует в составе исследуемого программного обеспечения.

В ходе анализа исходного кода была изучена функция `sqlite3KeyInfoFromExprList`, а также связанные с обработкой выражений и построением структуры `KeyInfo` участки реализации. Дополнительно исследовались функции обработки SQL-запросов и соответствующие фрагменты машинного кода `libtmessages.49.so`.

Исправление связано с корректной обработкой случая `SELECT DISTINCT ... ORDER BY`, когда все элементы результирующего набора являются константами, а количество элементов результирующего набора превышает количество элементов `ORDER BY`. Исправление присутствует в SQLite 3.41.2. В качестве первичного источника исправления используется запись SQLite Fossil с идентификатором `12ad822d9b827777`.

Таким образом, на уровне состава программного обеспечения Telegram содержит версию SQLite, в которой присутствует рассматриваемая уязвимость. Однако наличие уязвимого кода не означает, что её можно эксплуатировать через обычный интерфейс приложения. Для эксплуатации требуется возможность передать произвольный SQL-запрос, а наличие такой возможности у внешнего пользователя в рамках стандартного сценария работы Telegram не подтверждено.

Поэтому корректная оценка результата следующая: **уязвимый компонент SQLite присутствует; бинарный файл содержит версию из диапазона затронутых выпусков; практическая достижимость уязвимого участка через интерфейсы Telegram не установлена**.

## 1.6. Выводы по исследованию Telegram

В результате статического анализа APK-файла Telegram версии 13.2.2 были исследованы четыре уникальные нативные библиотеки для архитектуры ARM64. Установлено, что приложение использует сторонние компоненты AndroidX DataStore, Google ML Kit Language Identification, TensorFlow Lite, FFmpeg, SQLite, WebRTC, Opus и Opusfile.

Для AndroidX DataStore и SQLite удалось получить особенно сильные подтверждения идентификации посредством сравнения контрольных сумм с официальными реализациями. Версия FFmpeg установлена по идентификаторам входящих в него библиотек. Для ML Kit подтверждена версия 17.0.6 на основании графа Gradle-зависимостей. Для TensorFlow Lite, WebRTC, Opus и Opusfile факт присутствия подтверждён, однако точные версии не установлены.

Анализ динамических зависимостей показал, что значительная часть сторонних компонентов не представлена отдельными разделяемыми библиотеками, а включена в состав `libtmessages.49.so` посредством статической компоновки или компиляции исходных файлов.

В рамках исследования уязвимостей рассмотрена CVE-2025-7458, связанная с SQLite. Установлено, что Telegram содержит SQLite версии 3.39.3, входящей в диапазон затронутых версий. Следовательно, уязвимый код присутствует в составе приложения. Вместе с тем возможность эксплуатации через стандартные интерфейсы Telegram не доказана, поскольку для атаки требуется выполнение произвольного SQL-запроса.

___
Листинг позволяет сделать достаточно уверенный вывод: **в исследуемой `libtmessages.49.so` отсутствует исправление SQLite, добавленное в версии 3.41.2 для CVE-2025-7458**. Причём теперь это можно обосновать не только строкой версии `3.39.3`, но и анализом машинного кода.

## 1. Где находится проверка `WHERE_DISTINCTBY`

Рассмотрим участок `0x01433544–0x014335C0`:

```
0x01433544  ldr  x1, [x19, 0x10]
...
0x01433554  ldrsb w10, [x28, 0x16]
...
0x0143355c  strb w10, [x19, 0x49]
...
0x014335ac  ldr  w9, [x1]
0x014335b0  cmp  w9, w10
0x014335b4  b.ne 0x1433628
0x014335b8  mov  w9, 2
0x014335bc  strb w9, [x19, 0x4b]
0x014335c0  b    0x1433628
```

С учётом ранее установленного соответствия полей структуры `WhereInfo`:

- `x19` указывает на `WhereInfo`;
- `[x19 + 0x10]` содержит `pOrderBy`;
- `[x19 + 0x44]` содержит `wctrlFlags`;
- `[x19 + 0x49]` соответствует `nOBSat`;
- `[x19 + 0x4b]` соответствует `eDistinct`;
- `x28` указывает на текущий `WhereLoop`, а поле `[x28 + 0x16]` содержит `isOrdered`.

Таким образом, инструкции выполняют следующую логику:

1. Загружают `pOrderBy` из `WhereInfo`.
2. Получают значение `isOrdered`.
3. Записывают его в `nOBSat`.
4. При установленном соответствующем флаге `WHERE_DISTINCTBY` сравнивают `pOrderBy->nExpr` с `isOrdered`.
5. При равенстве устанавливают `eDistinct = WHERE_DISTINCT_ORDERED`.

Это соответствует первой части рассматриваемого участка исходного кода SQLite.

## 2. Как должно выглядеть исправление

В SQLite 3.41.2 после проверки `eDistinct` присутствует дополнительный фрагмент:

```
if( pWInfo->pSelect->pOrderBy
 && pWInfo->nOBSat > pWInfo->pSelect->pOrderBy->nExpr ){
  pWInfo->nOBSat = pWInfo->pSelect->pOrderBy->nExpr;
}
```

Его назначение: ограничить `nOBSat` числом выражений в `ORDER BY` исходного `SELECT`, если текущее значение `nOBSat` оказалось больше.

Именно этой проверки нет в исследуемом участке дизассемблированной функции. После установки `eDistinct` управление сразу переходит на `0x01433628`:

```
0x014335b8  mov  w9, 2
0x014335bc  strb w9, [x19, 0x4b]
0x014335c0  b    0x1433628
```

Кроме того, в остальной части функции нет соответствующей последовательности, которая сравнивала бы `nOBSat` с количеством выражений в `pSelect->pOrderBy` и ограничивала его этим значением. В частности, запись в `nOBSat` в рассматриваемой ветви выполняется непосредственно из `isOrdered`, а не из результата такого ограничения. Вставленный текст

## 3. Вывод по CVE-2025-7458

Полученные результаты позволяют сформулировать следующий вывод:

|Проверка|Результат|
|---|---|
|Наличие SQLite|Подтверждено|
|Версия SQLite|3.39.3|
|Соответствие исходного файла официальной SQLite 3.39.3|Подтверждено совпадением SHA3-256|
|Наличие `wherePathSolver()` в бинарном файле|Подтверждено сопоставлением структуры и логики функции|
|Наличие исправления из SQLite 3.41.2|Не обнаружено|
|Наличие уязвимого кода|Подтверждено с высокой степенью уверенности|

**Следовательно, `libtmessages.49.so` содержит уязвимую версию SQLite, не включающую рассматриваемое исправление.** Это позволяет считать бинарный компонент уязвимым к CVE-2025-7458 на уровне реализации.

Однако здесь важно разделять наличие уязвимого кода и возможность его эксплуатации. Для эксплуатации CVE требуется возможность передать SQLite специально сформированный SQL-запрос. Сам факт наличия уязвимой библиотеки в Telegram не доказывает, что удалённый злоумышленник может вызвать соответствующий сценарий через штатный интерфейс приложения.

Вердикт: **Уязвимый компонент обнаружен; наличие доступного через штатный интерфейс приложения пути эксплуатации не подтверждено.**
