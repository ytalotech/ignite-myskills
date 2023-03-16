# Detox Setup

Documentação complementar

### 1. Detox CLI e Detox no projeto:

```bash
npm install -g detox-cli

ou

yarn add detox -D
```

\*Observação: é requerido o Jest instalado: **yarn add jest -D\***

### 2. Na pasta Android > build.gradle

```java
allprojects {
    repositories {
				...
        maven {
            // Artefados do Detox para o node_module
            url "$rootDir/../node_modules/detox/Detox-android"
        }
    }
}
```

### 3. Na pasta Android > App > build.gradle.

```java
android {
    defaultConfig {
				...
        // Novas dependencias para rodar os testes.
        testBuildType System.getProperty('testBuildType', 'debug')
        testInstrumentationRunner 'androidx.test.runner.AndroidJUnitRunner'
    }
}
```

No mesmo arquivo, adicione:

```java
dependencies {
		// Novas dependencias para rodar os testes.
    androidTestImplementation('com.wix:detox:+') { transitive = true }
    androidTestImplementation 'junit:junit:4.12'
}
```

### 4. Setup de teste

Agora, vamos criar um arquivo chamado **DetoxTest.java** no seguinte caminho:

**_android/app/src/androidTest/java/com/[nome_do_package]/DetoxTest.java_**

```java
package **com.package**; // Trocar pelo no do Projeto.

import com.wix.detox.Detox;
import com.wix.detox.config.DetoxConfig;

import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

import androidx.test.ext.junit.runners.AndroidJUnit4;
import androidx.test.filters.LargeTest;
import androidx.test.rule.ActivityTestRule;

@RunWith(AndroidJUnit4.class)
@LargeTest
public class DetoxTest {
    @Rule
    public ActivityTestRule<MainActivity> mActivityRule = new ActivityTestRule<>(MainActivity.class, false, false);

    @Test
    public void runDetoxTests() {
        DetoxConfig detoxConfig = new DetoxConfig();
        detoxConfig.idlePolicyConfig.masterTimeoutSec = 90;
        detoxConfig.idlePolicyConfig.idleResourceTimeoutSec = 60;
        detoxConfig.rnContextLoadTimeoutSec = (**com.package**.BuildConfig.DEBUG ? 180 : 60);

        Detox.runTests(mActivityRule, detoxConfig);
    }
}
```

### 5. Iniciando o Detox

```bash
yarn detox init -r jest
```

### 6. Atualizando o arquivo .detoxrc.json

```json
{
  "testRunner": "jest",
  "runnerConfig": "e2e/config.json",
  "configurations": {
    "android.emu.debug": {
      "type": "android.emulator",
      "binaryPath": "android/app/build/outputs/apk/debug/app-debug.apk",
      "build": "cd android && ./gradlew assembleDebug assembleAndroidTest -DtestBuildType=debug && cd ..",
      "device": {
        "avdName": "Pixel_3a_API_29"
      }
    }
  }
}
```

### 7. Gerando a build para teste

```bash
**No Android:**
yarn detox build -c android.emu.debug

**No iOS:**
yarn detox build -c ios.sim.debug
```

### 8. Executando testes com Detox
