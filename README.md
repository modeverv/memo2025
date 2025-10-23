# memo2025

https://claude.ai/share/a6760924-fe54-49d5-b31d-f112629a168f

```
plugins {
    id 'java'
}

group = 'org.example'
version = '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.apache.logging.log4j:log4j-to-slf4j:2.20.0'
    implementation 'org.slf4j:slf4j-nop:2.0.9'
    implementation 'org.apache.poi:poi-ooxml:5.4.1'
    implementation 'org.apache.poi:poi:5.4.1'
}

test {
    useJUnitPlatform()
}

tasks.register('copyDependencies', Copy) {
    from configurations.runtimeClasspath
    into 'build/jpackage-input'
}

tasks.register('createExecutableJar', Jar) {
    archiveFileName = 'myapp.jar'
    destinationDirectory = file('build/libs')

    manifest {
        attributes 'Main-Class': 'com.example.MainClass'
    }

    from sourceSets.main.output
}

tasks.register('prepareJpackage', Copy) {
    dependsOn 'createExecutableJar', 'copyDependencies'
    from 'build/libs/myapp.jar'
    into 'build/jpackage-input'
}

tasks.register('createMinimalRuntime', Exec) {
    dependsOn 'prepareJpackage'
    doFirst {
        delete 'build/minimal-runtime'
    }

    commandLine 'jlink',
            '--add-modules', 'java.base,java.xml,java.logging',  
            '--output', 'build/minimal-runtime',
            '--strip-debug',
            '--no-header-files',
            '--no-man-pages',
            '--compress', '2',
            '--strip-native-commands'
}

tasks.register('jpackageWindowsMinimal', Exec) {
    dependsOn 'createMinimalRuntime'

    commandLine 'jpackage',
            '--input', 'build/jpackage-input',
            '--name', 'xlsxEncrypter',
            '--main-jar', 'myapp.jar',
            '--main-class', 'org.example.Main',
            '--type', 'exe',
            '--dest', 'build/dist/windows',
            '--runtime-image', 'build/minimal-runtime',
            '--app-version', '1.0.0',
            '--vendor', 'Your Company'
}

tasks.register('jpackageMacMinimal', Exec) {
    dependsOn 'createMinimalRuntime'

    commandLine 'jpackage',
            '--input', 'build/jpackage-input',
            '--name', 'xlsxEncrypter',
            '--main-jar', 'myapp.jar',
            '--main-class', 'org.example.Main',
            '--type', 'dmg',
            '--dest', 'build/dist/mac',
            '--runtime-image', 'build/minimal-runtime',
            '--app-version', '1.0.0',
            '--mac-package-name', 'xlsxEncrypter'
}

tasks.register('jpackageLinuxMinimal', Exec) {
    dependsOn 'createMinimalRuntime'

    commandLine 'jpackage',
            '--input', 'build/jpackage-input',
            '--name', 'xlsxEncrypter',
            '--main-jar', 'myapp.jar',
            '--main-class', 'org.example.Main',
            '--type', 'deb',
            '--dest', 'build/dist/linux',
            '--runtime-image', 'build/minimal-runtime',
            '--app-version', '1.0.0',
            '--linux-package-name', 'xlsxEncrypter'
}
package org.example;

import org.apache.poi.openxml4j.opc.OPCPackage;
import org.apache.poi.openxml4j.opc.PackageAccess;
import org.apache.poi.poifs.crypt.EncryptionInfo;
import org.apache.poi.poifs.crypt.EncryptionMode;
import org.apache.poi.poifs.crypt.Encryptor;
import org.apache.poi.poifs.filesystem.POIFSFileSystem;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;

public class Main {
    public static void main(String[] args) {
        if (args.length != 3) {
            System.exit(1);
        }

        String inputFilePath = args[0];
        String outputFilePath = args[1];
        String password = args[2];

        try (POIFSFileSystem fs = new POIFSFileSystem()) {
            EncryptionInfo info = new EncryptionInfo(EncryptionMode.agile);
            Encryptor encryptor = info.getEncryptor();
            encryptor.confirmPassword(password);

            try (OPCPackage opc = OPCPackage.open(new File(inputFilePath), PackageAccess.READ_WRITE);
                 OutputStream os = encryptor.getDataStream(fs)) {
                opc.save(os);
            }

            try (FileOutputStream fos = new FileOutputStream(outputFilePath)) {
                fs.writeFilesystem(fos);
            }
        } catch (Exception e) {
            e.printStackTrace();
            System.exit(1);
        }
        System.exit(0);
    }
}
```





