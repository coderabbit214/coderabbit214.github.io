# 数据结构与算法---稀疏数组


# 数据结构与算法---稀疏数组

## 概念

作用：当一个数组中大部分元素为0，或者为同一个值的数组时，可以使用稀疏数组来保存该数组。

[![tnmGuj.th.png](https://s1.ax1x.com/2020/05/29/tnmGuj.th.png)](https://imgchr.com/i/tnmGuj)

> 第一行 存储 原始二维数组有几行几列几个有效数值
>
> 第二行及以后存储有效数值以及位置

## 代码实现

```java
package com.jsh.DataStructures;

import java.io.*;

/*
    Mr.J
    稀疏数组
    2020/05/28
 */
public class SparseArray {
    public static void main(String[] args) throws IOException {
//      创建一个原始的二维数组 11*11
//      0:表示没有棋子，1表示黑子，2表示蓝子
        int chessArr1[][] = new int[11][11];
        chessArr1[1][2] = 1;
        chessArr1[2][3] = 2;
//      输出原始二维数组
        System.out.println("原始二维数组");
        for (int[] row:chessArr1){
            for (int data:row){
                System.out.print(data+"\t");
            }
            System.out.println();
        }

//      二维数组 => 稀疏数组
//      1.遍历二维数组 得到非零数据的个数
        /*
           chessArr1.length 获取行数---11行
           chessArr1[0].length 获取列数---11列
         */
        int sum = 0;
        for (int i = 0;i < chessArr1.length;i++){
            for (int j = 0;j < chessArr1[0].length;j++) {
                if (chessArr1[i][j] != 0){
                    sum++;
                }
            }
        }
        System.out.println("sum="+sum);
//      2.创建对应的稀疏数组
        int sparseArr[][] = new int[sum+1][3];
//      给稀疏数组赋值
//      第一行
        sparseArr[0][0] = chessArr1.length;
        sparseArr[0][1] = chessArr1[0].length;
        sparseArr[0][2] = sum;
//      其他
//      遍历二维数组，将非零的值放在sparseArr中
        int count = 0;//count 用于记录是第几个非零数据
        for (int i = 0;i < chessArr1.length;i++){
            for (int j = 0;j < chessArr1[0].length;j++) {
                if (chessArr1[i][j] != 0){
                    count++;
                    sparseArr[count][0] = i;
                    sparseArr[count][1] = j;
                    sparseArr[count][2] = chessArr1[i][j];
                }
            }
        }
//      将稀疏数组写入文件中
//        File chess = new File("D://chess.data");
//        FileWriter out = new FileWriter(chess);
//      输出稀疏数组
        System.out.println("得到的稀疏数组为---");
        for (int[] row:sparseArr){
            for (int data:row){
                System.out.print(data+"\t");
//                out.write(data+"\t");
            }
            System.out.println();
//            out.write("\n");
        }
//        out.close();
//        BufferedReader in = new BufferedReader(new FileReader(chess));
//        String line;//一行数据
//        int row=0;
//        //逐行读取，并将每个数组放入到数组中
//        while ((line = in.readLine()) != null){
//            String[] temp = line.split("\t");
//            for(int j=0;j<temp.length;j++){
//                sparseArr[row][j] = Integer.parseInt(temp[j]);
//            }
//            row++;
//        }
//        in.close();
//
//        System.out.println("得到的稀疏数组为---");
//        for (int[] raw:sparseArr){
//            for (int data:raw){
//                System.out.print(data+"\t");
//            }
//            System.out.println();
//        }
//      稀疏数组 => 二维数组
//      1.先读取稀疏数组的第一行，根据的一行的数据，创建原始的二维数组
        int chessArr2[][] = new int[sparseArr[0][0]][sparseArr[0][1]];
//      2.读取第二行及以后的数据，赋给原始二维数组
        for (int i = 1;i < sparseArr.length;i++){
            chessArr2[sparseArr[i][0]][sparseArr[i][1]] = sparseArr[i][2];
        }
        //输出恢复后的二维数组
        System.out.println("恢复后的二维数组");
        for (int[] raw:chessArr2){
            for (int data:raw){
                System.out.print(data+"\t");
            }
            System.out.println();
        }
    }
}

```


