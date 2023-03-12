# 用JAVA实现文件切割和MD5加密

## 文件切割小文本

思路：

1. 首先按行读取文本数据，然后拿文本的总行数/切割数,然后将得到的行数按行循环输送到自定义文本中

代码实现：

~~~java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.io.PrintWriter;

public class FileCut {
    public FileCut() {
    }

    public static void main(String[] args) throws IOException {
        fileCut(args[0], args[1], Integer.parseInt(args[2]));
    }

    public static void fileCut(String from, String to, int subFileNum) throws IOException {
        File fromFile = new File(from);
        if (!fromFile.exists()) {
            throw new IOException(fromFile + "不存在");
        } else if (!fromFile.isFile()) {
            throw new IOException(fromFile + "不是文件");
        } else {
            BufferedReader br = new BufferedReader(new FileReader(fromFile));
            String str = null;
            Integer fileRows = getROws(fromFile);
            int subRows = (int)Math.ceil(1.0D * (double)fileRows / (double)subFileNum);

            for(int i = 0; i < subFileNum; ++i) {
                PrintWriter pw = new PrintWriter(to + "-" + i + ".del");
                int row = 0;
                System.out.println("生成第: " + (i + 1) + "个文件");

                while(row < subRows) {
                    if ((str = br.readLine()) == null) {
                        pw.flush();
                        pw.close();
                        break;
                    }

                    pw.println(str);
                    ++row;
                }

                pw.flush();
                pw.close();
            }

            br.close();
            System.out.println("文件拆分完成");
        }
    }

    public static Integer getROws(File file) throws IOException {
        int rows = 0;

        for(BufferedReader br = new BufferedReader(new FileReader(file)); br.readLine() != null; ++rows) {
        }

        return rows;
    }
}

~~~

## MD5实现

思路：

1. 用到MessageDigest这个类，将每行读取的数据以字节数组形式传入digest方法
2. 返回得到16长度的字节数组，将这个字节数组转换成16进制的的字符串，确保转换的字符串不要太长，控制值在0~256之间，因为16*16=256，相当于16进制的FF

代码 ：

~~~java


import java.io.*;
import java.nio.charset.StandardCharsets;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class ToMD5 {

    public static void main(String[] args) throws Exception {
        if(args != null || args.length != 2){
            throw new IllegalArgumentException("参数需要两个");
        }
        String to = args[0];
        String from = args[1];
        File fromFile = new File(to);
        checkUpfilePath(fromFile);
        checkUpFile(fromFile);
        File toFile = new File(from);
        checkUpfilePath(toFile);
        writeWithMd5(fromFile,toFile);

    }

    public static void checkUpfilePath(File file) throws IOException {
        if (!file.getParentFile().isDirectory()) {
            throw new IOException(file + "该目录不存在");
        }

    }

    public static void checkUpFile(File file) throws IOException {
        if (!file.isFile()) {
            throw new IOException(file + "不是文件");
        }
    }

    public static void writeWithMd5(File fromFile, File toFile) throws IOException, NoSuchAlgorithmException {
        BufferedReader bf = new BufferedReader(new FileReader(fromFile));
        PrintWriter pw = new PrintWriter(toFile);
        String line = null;
        while ((line = bf.readLine()) != null) {
           String line1 = line.trim().replaceAll("\\s+", ",").replaceAll("\\t{1,}",",");
            String lineToMd5 = toMd5(line);
            String res = "\"" + line1 + "\"" + "," + "\"" + lineToMd5 + "\"";
            System.out.println(res);
            pw.println("\"" + line1 + "\"" + "," + "\"" + lineToMd5 + "\"");
        }
        pw.flush();
        pw.close();
    }

    public static String toMd5(String line) throws NoSuchAlgorithmException {
        MessageDigest md5 = MessageDigest.getInstance("MD5");
        StringBuilder sb = new StringBuilder();
        byte[] byteArr = md5.digest(line.getBytes(StandardCharsets.UTF_8));
        for (byte bt : byteArr) {
            //控制bt在0~256之间，这样转换到的16进制最长就是0~ff，md5返回的数组长度是16，最多32位
            if (bt < 0) {
                bt += 256;
            }
            sb.append(Integer.toHexString(bt));
        }
        return sb.toString();
    }
}

~~~

