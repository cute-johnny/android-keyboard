# Large dictionaries (format version 203)

Stock AOSP/FUTO binary dictionaries (format v202) store internal pointers in at
most 3 bytes, so a `.dict` file can't be larger than 16 MiB. The stock
`dicttool` doesn't check this. It writes truncated pointers, and the keyboard
then shows garbage suggestions and crashes during swipe typing.

This fork adds **format version 203**. It has the same layout as v202, except
that the "three bytes" address type is stored in **4 bytes**. The native reader
in this fork (`native/jni/src/dictionary/...`) loads v203. Stock FUTO rejects
v203 files as an unknown version instead of misreading them.

## Building a v203 dictionary

`src/` contains two AOSP encoder classes (Apache-2.0, from android.googlesource.com
LatinIME master). Only `BinaryDictEncoderUtils.java` is modified. `Ver2DictEncoder.java`
is unchanged and is compiled alongside it. Both are put on the classpath ahead of the
stock `dicttool_aosp.jar`:

```sh
# dicttool_aosp.jar: https://codeberg.org/Helium314/aosp-dictionaries (dicttool_aosp.jar)
mkdir -p cls
javac -cp dicttool_aosp.jar -d cls src/com/android/inputmethod/latin/makedict/*.java
java -Xmx6g -Ddicttool.large=true \
     -cp cls:dicttool_aosp.jar com.android.inputmethod.latin.dicttool.Dicttool \
     makedict -s wordlist.combined -d main_xx.dict
```

Without `-Ddicttool.large=true`, the patched encoder writes normal v202 files.
If such a file would exceed 16 MiB, it fails with an error instead of writing a
corrupt dictionary.
