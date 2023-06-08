# Excelx C++ Library

该库的作用是在代码中实现对excel文档(.xlsx格式)的一些操作
- [Wordx](https://github.com/lixuan998/Wordx)


## 功能

 - [x] 添加表单
 - [x] 替换文字标签
 - [x] 向单元格中插入图片
 - [x] 向单元格中插入文字
 - [x] 从标签处循环插入图片
 - [x] 从标签处循环插入文字

## 依赖
- OpenCV (建议4.5.5版本)
- Qt (建议5.12.12版本)
- CMake (3.5以上版本)

## 安装
- 下载本库
- 将本库文件夹移动到你的项目目录中
- 在你的项目的CMakeLists.txt中添加 
``` CMake
  set(XML_MODEL_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/excelx/xml_models)
  set(XML_MODEL_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/xml_models)

  file(
    GLOB
    HEADER_DIR_SET
    LIST_DIRECTORIES true
    ${CMAKE_CURRENT_SOURCE_DIR}/excelx/include/*
)

  execute_process(COMMAND ${CMAKE_COMMAND} -E copy_directory  "${XML_MODEL_SOURCE_DIR}" "${XML_MODEL_BINARY_DIR}")

  add_subdirectory(excelx)

  target_include_directories(${YOUR_PROJECT} PRIVATE ${HEADER_DIR_SET})#也可以不用这行，但是在代码中头文件应改成#include"../include/Excelx/excelop.h"

  target_link_libraries(${YOUR_PROJECT} Excelx)#一般放在最后
```


##使用

``` C++
#include "excelop.h"
#include <QApplication>

/*
    @brief 本函数的作用为测试添加表单的功能
    @param[in] op ExcelOp 类的引用
*/
void TestAddSheet(ExcelOp &op);

/*
    @brief 本函数的作用为测试写入单元格的功能
    @param[in] op ExcelOp 类的引用
*/
void TestWriteCell(ExcelOp &op);

/*
    @brief 本函数的作用为测试插入图片的功能
    @param[in] op ExcelOp 类的引用
*/
void TestDrawCell(ExcelOp &op);

/*
    @brief 本函数的作用为测试替换文字的功能
    @param[in] op ExcelOp 类的引用
*/
void TestRepacement(ExcelOp &op);

/*
    @brief 本函数的作用为测试批量插入文字
    @param[in] op ExcelOp 类的引用
*/
void TestWriteBatched(ExcelOp &op);

/*
    @brief 本函数的作用为测试批量插入图片
    @param[in] op ExcelOp 类的引用
*/
void TestDrawBatch(ExcelOp &op);

QString test_file_path = "../";
QString test_material_path = "../demo_files/";

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);
    ExcelOp op(test_file_path + "test_excel.xlsx", test_file_path + "results.xlsx");
    op.open();

    TestAddSheet(op);
    TestWriteCell(op);
    TestDrawCell(op);
    TestRepacement(op);
    TestWriteBatched(op);
    TestDrawBatch(op);

    op.close();
    return 0;
}

void TestAddSheet(ExcelOp &op)
{
    op.AddSheet(1);
    op.AddSheet(-1);
    op.AddSheet();
}

void TestWriteCell(ExcelOp &op)
{
    //读取表单1中Z1~Z4单元格的样式信息
    QString s1 = op.GetCellAttribute(1, "Z1", "s");
    QString s2 = op.GetCellAttribute(1, "Z2", "s");
    QString s3 = op.GetCellAttribute(1, "Z3", "s");
    QString s4 = op.GetCellAttribute(1, "Z4", "s");

    //读取表单1中Z1~Z4单元格的高度信息
    QString height1 = op.GetCellHeight(1, "Z1");
    QString height2 = op.GetCellHeight(1, "Z2");
    QString height3 = op.GetCellHeight(1, "Z3");
    QString height4 = op.GetCellHeight(1, "Z4");
    

    QString test_arr[]{"Text1", "Text2", "Text3", "Text4", "Test5"};
    QString cells[] = {"A", "B", "C", "D", "E", "F"};
    QString s[] = {s1, s2, s3, s4};

    QString heights[] = {height1, height2, height3, height4};

    //循环使用WriteCell写入单元格
    for(int i = 0; i < 6; ++ i)
    {
        for(int j = 0; j < 5; ++ j)
        {
            if(j != 4) op.WriteCell(1, cells[i] + QString::number(2 * j + 1), test_arr[j], s[j], heights[j]);
            else op.WriteCell(1, cells[i] + QString::number(2 * j + 1), test_arr[j]);
        }
    }
}

void TestRepacement(ExcelOp &op)
{
    op.ReplaceSharedStringText("Style1", "样式一");
    op.ReplaceSharedStringText("Style2", "样式二");
    op.ReplaceSharedStringText("Style3", "样式三");
    op.ReplaceSharedStringText("Style4", "样式四");
}

void TestWriteBatched(ExcelOp &op)
{
    Info info, info_horizontal;

    //测试垂直插入
    QString data[4][5] = {{"${writeBatch1}", "1", "11", "111", "1111"}, {"${writeBatch2}", "2", "22", "222", "222"}, {"${writeBatch3}", "3", "33", "333", "3333"}, {"${writeBatch4}", "4", "44", "444", "4444"}};
    for(int i = 0; i < 4; ++ i)
    {
        for(int j = 1; j < 5; ++ j)
        {
            //循环载入插入文字信息
            op.AddInfo(info, data[i][0], data[i][j]);
        }
    }

    op.WriteBatch(1, info);

    //测试水平插入
    QString data_horizontal[4][5] = {{"${writeBatch11}", "1", "11", "111", "1111"}, {"${writeBatch22}", "2", "22", "222", "222"}, {"${writeBatch33}", "3", "33", "333", "3333"}, {"${writeBatch44}", "4", "44", "444", "4444"}};
    for(int i = 0; i < 4; ++ i)
    {
        for(int j = 1; j < 5; ++ j)
        {
            //循环载入插入文字信息
            op.AddInfo(info_horizontal, data_horizontal[i][0], data_horizontal[i][j]);
        }
    }

    op.WriteBatch(1, info_horizontal, HORIZONTAL);
}

void TestDrawCell(ExcelOp &op)
{
    QString img_path = test_material_path;
    op.DrawCell(1, img_path + "steel1.jpeg", "0", "12", "1", "14");
    op.DrawCell(1, img_path + "steel2.jpeg", "2", "12", "3", "14");
    op.DrawCell(1, img_path + "steel3.jpeg", "4", "12", "5", "14");
    op.DrawCell(1, img_path + "steel4.jpeg", "6", "12", "7", "14");
}

void TestDrawBatch(ExcelOp &op)
{
    Info info, info_horizontal;
    QString img_path = test_material_path;
    //测试垂直插入
    QString data[4][5] = {{"${image1}", img_path + "steel1.jpeg", img_path + "steel1.jpeg", img_path + "steel1.jpeg", img_path + "steel1.jpeg"}, 
                          {"${image2}", img_path + "steel2.jpeg", img_path + "steel2.jpeg", img_path + "steel2.jpeg", img_path + "steel2.jpeg"}, 
                          {"${image3}", img_path + "steel3.jpeg", img_path + "steel3.jpeg", img_path + "steel3.jpeg", img_path + "steel3.jpeg"}, 
                          {"${image4}", img_path + "steel4.jpeg", img_path + "steel4.jpeg", img_path + "steel4.jpeg", img_path + "steel4.jpeg"}};
    for(int i = 0; i < 4; ++ i)
    {
        for(int j = 1; j < 5; ++ j)
        {
            //循环载入插入图片信息
            op.AddInfo(info, data[i][0], data[i][j]);
        }
    }

    op.DrawBatch(1, info, 1000);

    //测试水平插入
    QString data_horizontal[4][5] = {{"${image11}", img_path + "steel1.jpeg", img_path + "steel1.jpeg", img_path + "steel1.jpeg", img_path + "steel1.jpeg"}, 
                          {"${image22}", img_path + "steel2.jpeg", img_path + "steel2.jpeg", img_path + "steel2.jpeg", img_path + "steel2.jpeg"}, 
                          {"${image33}", img_path + "steel3.jpeg", img_path + "steel3.jpeg", img_path + "steel3.jpeg", img_path + "steel3.jpeg"}, 
                          {"${image44}", img_path + "steel4.jpeg", img_path + "steel4.jpeg", img_path + "steel4.jpeg", img_path + "steel4.jpeg"}};
    for(int i = 0; i < 4; ++ i)
    {
        for(int j = 1; j < 5; ++ j)
        {
            //循环载入插入图片信息
            op.AddInfo(info_horizontal, data_horizontal[i][0], data_horizontal[i][j]);
        }
    }

    op.DrawBatch(1, info_horizontal, 1000, HORIZONTAL);
}
```

## API
```
#include "excelop.h"
```
 - bool open(QString src_model_path = "", QString dst_export_path = "");
 - void close();
 - bool AddSheet(int sheet_model_sn = 0);
 - bool WriteCell(int sheet_sn, QString cell_sn, QString text, QString cell_style = "0", QString cell_height = "12.8");
 - bool DrawCell(int sheet_sn, QString image_path, QString from_col, QString from_row, QString to_col, QString to_row, double times = DEFAULT_TIME);
 - int ReplaceSharedStringText(QString mark, QString text);
 - bool WriteBatch(int sheet_sn, Info info, int direction = VERTICAL);
 - bool DrawBatch(int sheet_sn, Info info, double times = DEFAULT_TIME, int direction = VERTICAL);
 - void AddInfo(Info &info, QString label, QString text);
 - void AddInfo(Info &info, QString label, cv::Mat mat);
 - int CountSheets();
 - QString GetCellAttribute(int sheet_sn, QString cell_sn, QString attribute_label);
 - QString GetCellHeight(int sheet_sn, QString cell_sn);

