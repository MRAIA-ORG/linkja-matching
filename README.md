# linkja-matching
Code to match patients across multiple hospitals

## Building
linkja-matching was tested under Java JDK 25 (specifically [Adoptium Temurin JDK](https://adoptium.net/temurin/releases)). It can be opened from within an IDE like Eclipse or IntelliJ IDEA and compiled, or compiled from the command line using [Maven](https://maven.apache.org/).

You can build linkja-matching via Maven:

`mvn clean package`

This compiles the code and puts an executable JAR with all dependency JARs in the `target` folder, e.g. `target/Matching-1.3.0-jar-with-dependencies.jar`. The project has no unit tests yet.

The GUI uses JavaFX, and JavaFX has one set of JARs per platform. The JAR you build holds the JavaFX files of one platform only, so `--gui` runs only on that platform. Command-line matching and `--decrypt` run from any of the JARs. By default the JAR gets the JavaFX files of the machine you build on. Linux and Windows on arm64 have no default. On Linux arm64, name `linux-aarch64`. JavaFX has no Windows arm64 build, so `--gui` does not run there. To build for another platform, name it with `javafx.classifier`, e.g. for an Apple Silicon Mac:

`mvn clean package -Djavafx.classifier=mac-aarch64`

The values are `win` (Windows x64), `linux` (Linux x64), `mac` (macOS x86_64) and `mac-aarch64` (macOS arm64).

CI runs the build on each pull request. See `.github/workflows/ci.yml`.

## Program Use
You can run the executable JAR file using the standard Java command:
`java -jar Matching-1.3.0-jar-with-dependencies.jar`

The program has two modes: command line, and a GUI.  By default it will run as a command line application.  If you would like to use the GUI, specify `--gui` from the command line.

### Decryption

You can run the executable JAR file (the one including `with-dependencies` in the file name) using the standard Java command:
`java -jar <JAR path>`

For decryption, linkja-matching uses a special C library (.dll/.dylib/.so, depending on your operating system).  You will need to tell Java where to find this library when you try to run the program.  Otherwise, you will get an error:

```
Exception in thread "main" java.lang.UnsatisfiedLinkError: no linkjacrypto in java.library.path:
```

The library may be placed in any directory found by the Java library path.  If you would like to specify the library, you can include the `-Djava.library.path=` option when running the program.
This can be the same directory as the linkja-matching JAR file (e.g., `-Djava.library.path=.`).

Decryption needs only the public build of the library, because it does not call `createSecureHash`. The library files in the `lib` folder of this repository are an old build: the macOS one does not run on Apple Silicon. To compile the public build, use the `develop` branch of [linkja-crypto](https://github.com/MRAIA-ORG/linkja-crypto). It needs OpenSSL 3, CMake and cmocka (e.g. `brew install openssl@3 cmake cmocka`). From the linkja-crypto folder on macOS:

```
export JAVA_HOME="$(/usr/libexec/java_home -F -v 25 2>/dev/null || brew --prefix openjdk@25)"
cmake . -DINCLUDE_SECRETS=OFF -DCMAKE_BUILD_TYPE=Release -DOPENSSL_ROOT_DIR="$(brew --prefix openssl@3)"
make linkjacrypto
```

On Linux, leave out `-DOPENSSL_ROOT_DIR`. The library is then in the `target` folder of linkja-crypto.

Note that where files are used for input, they can be specified as a relative or absolute path.

Decryption uses the following parameters, all of which are required:

```
--decrypt            Enables the decryption mode
--directory          The root directory of the project.  The program assumes the project structure
                     follows our convention of a `data/input` sub folder under which the encrypted files
                     are stored.  The private RSA key is in the root directory itself.
--prefix             The prefix of all encrypted file names (e.g., hashes)
--suffix             The suffix (extension) of all encrypted file names (e.g., .enc).
                     It is recommended to include the '.' in the extension.
--decryptionKey      The name of the RSA private key to decrypt the files.  This should be the private
                     part of the public key shared with each hashing site.
                     This is JUST the file name - the program looks for it directly in --directory.
```

**Example**

Given the following project structure:

```
C:\Projects\Project1\
                     private.key
                     data\
                          input\
                                hashes_1_Linkja Project_20200516142804.enc
                                hashes_2_Linkja Project_20200516142804.enc
```

The following command would be used to decrypt the two files. It writes one CSV file beside each `.enc` file:

```
java -Djava.library.path=. -jar Matching-1.3.0-jar-with-dependencies.jar --decrypt
  --directory C:\Projects\Project1 --prefix hashes --suffix .enc
  --decryptionKey private.key

```
