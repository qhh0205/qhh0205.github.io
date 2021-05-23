---
title: Java 解压 gzip 和 tar.gz 文件
date: 2021-05-23 11:35:53
categories: Java
tags: Java
---

在开发过程中有时候会需要解压 gzip 或者 tar.gz 文件，下面封装了一个工具类，可以解压 gzip 和 tar.gz 文件。

```java
package com.example.demo.common.utils;
/**
 * Created by qianghaohao on 2021/5/23
 */

import org.apache.commons.compress.archivers.tar.TarArchiveEntry;
import org.apache.commons.compress.archivers.tar.TarArchiveInputStream;
import org.apache.commons.compress.compressors.gzip.GzipCompressorInputStream;

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.zip.GZIPInputStream;

/**
 * @description:
 * @author: qianghaohao
 * @time: 2021/5/23
 */
public class FileUtils {
    private static final int BUFFER_SIZE = 1024;

    /**
     * 解压 gzip 文件
     *
     * @param input
     * @param output
     *
     */
    public static void decompressGzip(File input, File output) throws IOException {
        try (GZIPInputStream in = new GZIPInputStream(new FileInputStream(input))) {
            try (FileOutputStream out = new FileOutputStream(output)) {
                byte[] buffer = new byte[BUFFER_SIZE];
                int len;
                while((len = in.read(buffer)) != -1) {
                    out.write(buffer, 0, len);
                }
            }
        }
    }

    /**
     * 解压 tar.gz 文件到指定目录
     *
     * @param tarGzFile  tar.gz 文件路径
     * @param destDir  解压到 destDir 目录，如果没有则自动创建
     *
     */
    public static void extractTarGZ(File tarGzFile, String destDir) throws IOException {

        GzipCompressorInputStream gzipIn = new GzipCompressorInputStream(new FileInputStream(tarGzFile));
        try (TarArchiveInputStream tarIn = new TarArchiveInputStream(gzipIn)) {
            TarArchiveEntry entry;

            while ((entry = (TarArchiveEntry) tarIn.getNextEntry()) != null) {
                if (entry.isDirectory()) {
                    File f = new File(destDir + "/" + entry.getName());
                    boolean created = f.mkdirs();
                    if (!created) {
                        System.out.printf("Unable to create directory '%s', during extraction of archive contents.\n",
                                f.getAbsolutePath());
                    }
                } else {
                    int count;
                    byte [] data = new byte[BUFFER_SIZE];
                    FileOutputStream fos = new FileOutputStream(destDir + "/" + entry.getName(), false);
                    try (BufferedOutputStream dest = new BufferedOutputStream(fos, BUFFER_SIZE)) {
                        while ((count = tarIn.read(data, 0, BUFFER_SIZE)) != -1) {
                            dest.write(data, 0, count);
                        }
                    }
                }
            }
        }
    }
}
```

使用示例：
```java
    @Test
    public void decompressGizpTest() throws IOException {
        File input = new File("/xxx/output.gz");
        File output = new File("/xxx/output");
        FileUtils.decompressGzip(input, output);
    }

    @Test
    public void decompressTarGizpTest() throws IOException {
        File input = new File("/xxx/output.tar.gz");
        FileUtils.extractTarGZ(input, "/tmp/");
    }
```

