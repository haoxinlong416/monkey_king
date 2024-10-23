main.cpp

```c
#include <QApplication>  
#include "mainwindow.h"  
  
int main(int argc, char *argv[])  
{  
    QApplication a(argc, argv);  
    MainWindow w;  
    w.show();  
    return a.exec();  
}
```

 **mainwindow.h** 

```
#ifndef MAINWINDOW_H  
#define MAINWINDOW_H  
  
#include <QMainWindow>  
#include <QTimer>  
#include <QNetworkAccessManager>  
#include <QNetworkReply>  
  
QT_BEGIN_NAMESPACE  
namespace Ui { class MainWindow; }  
QT_END_NAMESPACE  
  
class MainWindow : public QMainWindow  
{  
    Q_OBJECT  
  
public:  
    MainWindow(QWidget *parent = nullptr);  
    ~MainWindow();  
  
private slots:  
    void on_checkServerButton_clicked();  
    void onReplyFinished(QNetworkReply *reply);  
  
private:  
    Ui::MainWindow *ui;  
    QTimer *timer;  
    QNetworkAccessManager *networkManager;  
};  
  
#endif // MAINWINDOW_H
```

 **mainwindow.cpp** 

```
#include "mainwindow.h"  
#include "ui_mainwindow.h"  
#include <QDebug>  
#include <QVBoxLayout>  
#include <QPushButton>  
#include <QTextEdit>  
  
MainWindow::MainWindow(QWidget *parent)  
    : QMainWindow(parent)  
    , ui(new Ui::MainWindow)  
    , timer(new QTimer(this))  
    , networkManager(new QNetworkAccessManager(this))  
{  
    ui->setupUi(this);  
  
    // Create a simple layout with a button and a text edit  
    QWidget *centralWidget = new QWidget(this);  
    QVBoxLayout *layout = new QVBoxLayout(centralWidget);  
  
    QPushButton *checkServerButton = new QPushButton("Check Server", this);  
    QTextEdit *responseTextEdit = new QTextEdit(this);  
    responseTextEdit->setReadOnly(true);  
  
    layout->addWidget(checkServerButton);  
    layout->addWidget(responseTextEdit);  
  
    ui->setCentralWidget(centralWidget);  
  
    connect(checkServerButton, &QPushButton::clicked, this, &MainWindow::on_checkServerButton_clicked);  
    connect(timer, &QTimer::timeout, this, [this]() {  
        this->on_checkServerButton_clicked();  
    });  
    connect(networkManager, &QNetworkAccessManager::finished, this, &MainWindow::onReplyFinished);  
}  
  
MainWindow::~MainWindow()  
{  
    delete ui;  
}  
  
void MainWindow::on_checkServerButton_clicked()  
{  
    QUrl url("http://your-server-address/your-endpoint"); // Replace with your server's URL  
    QNetworkRequest request(url);  
    QNetworkReply *reply = networkManager->get(request);  
  
    // Optionally, you can set a timeout for the request  
    QTimer::singleShot(5000, [reply]() {  
        if (reply && !reply->isFinished()) {  
            reply->abort();  
        }  
    });  
}  
  
void MainWindow::onReplyFinished(QNetworkReply *reply)  
{  
    if (reply->error() == QNetworkReply::NoError) {  
        QString response = reply->readAll();  
        qDebug() << "Server response:" << response;  
  
        // Display the response in the text edit  
        QTextEdit *responseTextEdit = qobject_cast<QTextEdit*>(ui->centralWidget()->findChild<QTextEdit*>("qt_text_edit_child"));  
        if (responseTextEdit) {  
            responseTextEdit->append(response);  
        }  
    } else {  
        qDebug() << "Error:" << reply->errorString();  
    }  
  
    reply->deleteLater();  
}
```

1. **main.cpp**：创建并显示主窗口。

2. **mainwindow.h**：声明主窗口类，包括槽函数和成员变量。

3. mainwindow.cpp

   ：实现主窗口类。

   - 创建一个简单的布局，包含一个按钮和一个只读文本编辑框。
   - 连接按钮的点击信号到槽函数`on_checkServerButton_clicked`。
   - 使用`QNetworkAccessManager`发送HTTP GET请求到服务器。
   - 连接`QNetworkReply`的`finished`信号到槽函数`onReplyFinished`以处理响应。
   - 在`onReplyFinished`槽函数中，检查是否有错误并显示响应内容。

| 成员   | 具体工作           | 评分 |
| ------ | ------------------ | ---- |
| 杨泽宁 | 基础配置           | 112  |
| 陈嘉兴 | 文件配置           | 98   |
| 姜欢   | 查阅资料           | 98   |
| 王志华 | qt配置             | 98   |
| 郝鑫龙 | 网络配置           | 98   |
| 魏子越 | 文档编辑           | 98   |
| 王欣   | 整合修改，文档上传 | 98   |

