> 本Demo通过信号与槽的机制实现点击PushButton更改Label中的文本
![[Pasted image 20240729185634.png]]

mainwindow.h
```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H
#include <QMainWindow>

QT_BEGIN_NAMESPACE
namespace Ui {
class MainWindow;
}
QT_END_NAMESPACE

class MainWindow : public QMainWindow
{
    Q_OBJECT
    
public:
    MainWindow(QWidget *parent = nullptr);
    ~MainWindow();

private:
    Ui::MainWindow *ui;
    void on_button_clicked(); #在此声明on_button_clicked()函数 用于处理点击事件
};
#endif // MAINWINDOW_H

```

> main.cpp
```cpp
#include "mainwindow.h"

#include <QApplication>
#include <QLocale>
#include <QTranslator>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);

    // QTranslator translator;
    // const QStringList uiLanguages = QLocale::system().uiLanguages();
    // for (const QString &locale : uiLanguages) {
    //     const QString baseName = "img1_" + QLocale(locale).name();
    //     if (translator.load(":/i18n/" + baseName)) {
    //         a.installTranslator(&translator);
    //         break;
    //     }
    // }
    MainWindow w;
    w.show();
    return a.exec();
}

```

> mainwindow.cpp
```cpp
#include "mainwindow.h"
#include "./ui_mainwindow.h"
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    connect(ui->pushButton,&QPushButton::clicked,this,&MainWindow::on_button_clicked); #connect方法绑定PushButton和on_button_clicked函数
    
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::on_button_clicked() #在此实现on_button_clicked()方法
{
    ui->label->setText("clicked!"); 
}
```


### Qt 信号和槽机制

> [!NOTE]
> Qt 的信号和槽机制是一种用于对象间通信的机制。当一个对象（通常是UI组件）在某个事件发生时发出信号，另一个对象可以通过槽函数来响应这个信号。

> `connect` 方法用于将信号和槽连接起来。它的典型用法如下：
```cpp
	connect(sender, signal, receiver, slot);
```
- `sender`: 发出信号的对象。
- `signal`: 发送的信号。
- `receiver`: 接收信号的对象。
- `slot`: 处理信号的槽函数。