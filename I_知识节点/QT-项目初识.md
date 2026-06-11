# Qt

## 目录结构

### 一个完整的Qt项目通常包含以下文件和目录：

1. **源代码文件**：
   - `.cpp` 文件：包含项目的源代码实现。
   - `.h` 文件：包含项目的头文件声明。
2. **UI文件**：
   - `.ui` 文件：由Qt Designer创建的用户界面文件，描述了项目的界面布局和组件。
3. **资源文件**：
   - `.qrc` 文件：Qt资源文件，用于管理项目中使用的图像、字体、样式表等资源。
4. **构建文件**：
   - `.pro` 文件：Qt项目文件，描述了项目的构建设置和依赖关系。
   - `.pro.user` 文件（可选）：包含用户特定的构建设置，如编译器选项、运行配置等。
5. **其他辅助文件**：
   - `.h` 或 `.cpp` 文件：其他需要的头文件或源文件。
   - 其他资源文件（如图像、字体等）：项目中使用的其他资源文件。

项目的文件结构通常如下：

```css
ProjectName/
│
├── src/
│   ├── main.cpp
│   ├── MainWindow.cpp
│   └── OtherSourceFiles.cpp
│
├── include/
│   ├── MainWindow.h
│   └── OtherHeaderFiles.h
│
├── ui/
│   ├── MainWindow.ui
│   └── OtherUIFiles.ui
│
├── resources/
│   ├── images/
│   │   └── image1.png
│   ├── fonts/
│   │   └── font1.ttf
│   └── OtherResourceFiles
│
├── ProjectName.pro
├── ProjectName.pro.user
└── ProjectName.qrc

```

### main.cpp

```c++
#include <QApplication>
#include "MainWindow.h"

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    MainWindow w;
    w.show();
    return a.exec();
}

```

### MainWindow.cpp

```c++
#include "MainWindow.h"
#include "ui_MainWindow.h"

MainWindow::MainWindow(QWidget *parent) :
    QMainWindow(parent),
    ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}

MainWindow::~MainWindow()
{
    delete ui;
}

```

