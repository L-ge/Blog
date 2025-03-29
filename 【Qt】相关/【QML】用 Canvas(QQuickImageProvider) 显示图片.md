1. 大体功能：
- 频繁地往界面推送图片，帧率达到视频效果。
- 捕获画布上的鼠标事件和键盘事件。

2. 代码如下：
```cpp
// DrawImageInCanvas.pro 代码如下：
QT += quick

# You can make your code fail to compile if it uses deprecated APIs.
# In order to do so, uncomment the following line.
#DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    # disables all the APIs deprecated before Qt 6.0.0

SOURCES += \
        drawimagectrl.cpp \
        imageprovider.cpp \
        main.cpp

RESOURCES += qml.qrc

# Additional import path used to resolve QML modules in Qt Creator's code model
QML_IMPORT_PATH =

# Additional import path used to resolve QML modules just for Qt Quick Designer
QML_DESIGNER_IMPORT_PATH =

# Default rules for deployment.
qnx: target.path = /tmp/$${TARGET}/bin
else: unix:!android: target.path = /opt/$${TARGET}/bin
!isEmpty(target.path): INSTALLS += target

HEADERS += \
    drawimagectrl.h \
    imageprovider.h

LIBS += -lopencv_core \
        -lopencv_imgcodecs


// imageprovider.h 代码如下：
#ifndef IMAGEPROVIDER_H
#define IMAGEPROVIDER_H

#include <QObject>
#include <QQuickImageProvider>
#include "opencv2/core/core.hpp"

// 子类化 QQuickImageProvider
class ImageProvider : public QQuickImageProvider
{
public:
    explicit ImageProvider();

    // QQuickImageProvider interface (重写一个方法，也可以重新requestPixmap(),用于qml找图片提供者请求图片的格式)
    QImage requestImage(const QString &id, QSize *size, const QSize &requestedSize) override;
    void addImage(const cv::Mat &mat);

private:
    QImage m_img;
};

#endif // IMAGEPROVIDER_H


// imageprovider.cpp 代码如下：
#include "imageprovider.h"
#include "opencv2/core/mat.hpp"

ImageProvider::ImageProvider()
    : QQuickImageProvider(QQuickImageProvider::Image) // 必须初始化基类
{
}

QImage ImageProvider::requestImage(const QString &id, QSize *size, const QSize &requestedSize)
{
    return m_img;
}

void ImageProvider::addImage(const cv::Mat &mat)
{
    switch(mat.type())
    {
    case CV_8UC1:
        //QImage构造：数据，宽度，高度，每行多少字节，存储结构
        m_img = QImage((const unsigned char*)mat.data, mat.cols, mat.rows, mat.cols, QImage::Format_Grayscale8);
        break;
    case CV_8UC3:
        m_img = QImage((const unsigned char*)mat.data, mat.cols, mat.rows, mat.cols * 3, QImage::Format_RGB888);
        m_img = m_img.rgbSwapped(); //BRG转为RGB
        //Qt5.14增加了Format_BGR888
        //image = QImage((const unsigned char*)mat.data, mat.cols, mat.rows, mat.cols * 3, QImage::Format_BGR888);
        break;
    case CV_8UC4:
        m_img = QImage((const unsigned char*)mat.data, mat.cols, mat.rows, mat.cols * 4, QImage::Format_ARGB32);
        break;
    }
}


// drawimagectrl.h 代码如下：
#ifndef DRAWIMAGECTRL_H
#define DRAWIMAGECTRL_H

#include <QObject>
#include "imageprovider.h"

#define SINGLETON(x)     \
private: \
    x(x const&); \
    void operator=(x const&); \
public: \
static x* getInstance(QObject *parent = 0) \
{ \
   static bool first=true; \
   if ((parent == 0) && (first == true)) { qCritical("Incorrect Initialisation - no parent and first"); } \
   if ((parent != 0) && (first == false)) { qCritical("Incorrect Initialisation - parent and not first"); } \
   first = false; \
   static x *instance = new x(parent); \
   return instance; \
} \
private:

class DrawImageCtrl : public QObject
{
    Q_OBJECT
    SINGLETON(DrawImageCtrl)

public:
    explicit DrawImageCtrl(QObject *parent = nullptr);
    ~DrawImageCtrl();

    ImageProvider* getImageProvider() const { return m_pImageProvider; }

public slots:
    void sltTestDrawImage();

    // 辅助事件
    void sltWindowWidthChanged(qreal lfWindowWidth);
    void sltWindowHeightChanged(qreal lfWindowHeight);

    // 鼠标事件
    void sltMousePressed(qreal lfX, qreal lfY, quint32 nButtonValue);
    void sltMouseReleased(qreal lfX, qreal lfY, quint32 nButtonValue);

    // 键盘事件
    void sltKeyPressed(int nKey);
    void sltKeyReleased(int nKey);

signals:
    void sigImageUpdate(double lfImgWidth, double lfImgHeight, double lfFactor, double lfImgX, double lfImgY);

private:
    void showImage(const cv::Mat &mat);

private:
    // 界面属性
    qreal m_lfWindowWidth;
    qreal m_lfWindowHeight;
    double m_lfLastMouseX;
    double m_lfLastMouseY;

    // 快照属性
    double m_lfZoomFactor;
    double m_lfImgX;
    double m_lfImgY;

    ImageProvider* m_pImageProvider;
};

#endif // DRAWIMAGECTRL_H


// drawimagectrl.cpp 代码如下：
#include "drawimagectrl.h"
#include "opencv2/opencv.hpp"
#include <QTimer>

DrawImageCtrl::DrawImageCtrl(QObject *parent)
    : QObject(parent)
    , m_lfWindowWidth(0.0)
    , m_lfWindowHeight(0.0)
    , m_lfLastMouseX(-1)
    , m_lfLastMouseY(-1)
    , m_lfZoomFactor(1.0)
    , m_lfImgX(0.0)
    , m_lfImgY(0.0)
    , m_pImageProvider(new ImageProvider())
{

}

DrawImageCtrl::~DrawImageCtrl()
{
    if(nullptr != m_pImageProvider)
    {
        delete m_pImageProvider;
        m_pImageProvider = nullptr;
    }
}

void DrawImageCtrl::sltTestDrawImage()
{
    static int i = 0;
    QString sPicFileName = QString::fromUtf8("/home/xiaohuamao/mycode/DrawImageInCanvas/TestPic/%1.png").arg(i);
    if(++i >= 100)
        i = 0;
    cv::Mat image = cv::imread(sPicFileName.toStdString());
    showImage(image);
    QTimer::singleShot(10, this, &DrawImageCtrl::sltTestDrawImage);
}

void DrawImageCtrl::sltWindowWidthChanged(qreal lfWindowWidth)
{
    m_lfWindowWidth = lfWindowWidth;
}

void DrawImageCtrl::sltWindowHeightChanged(qreal lfWindowHeight)
{
    m_lfWindowHeight = lfWindowHeight;
}

void DrawImageCtrl::sltMousePressed(qreal lfX, qreal lfY, quint32 nButtonValue)
{
    qDebug() << QString::fromUtf8("sltMousePressed x:%1,y:%2,btn:%3")
                .arg(lfX).arg(lfY).arg(nButtonValue);
}

void DrawImageCtrl::sltMouseReleased(qreal lfX, qreal lfY, quint32 nButtonValue)
{
    qDebug() << QString::fromUtf8("sltMouseReleased x:%1,y:%2,btn:%3")
                .arg(lfX).arg(lfY).arg(nButtonValue);
}

void DrawImageCtrl::sltKeyPressed(int nKey)
{
    qDebug() << QString::fromUtf8("sltKeyPressed nKey:%1").arg(nKey);
}

void DrawImageCtrl::sltKeyReleased(int nKey)
{
    qDebug() << QString::fromUtf8("sltKeyReleased nKey:%1").arg(nKey);
}

void DrawImageCtrl::showImage(const cv::Mat &mat)
{
    cv::Mat image = mat.clone();
    m_pImageProvider->addImage(image);

    auto lfAspectRatio = 1. * image.cols / image.rows;
    auto lfAspectRatio2 = 1. * m_lfWindowWidth / m_lfWindowHeight;
    auto lfFactor = 1.0;
    if (lfAspectRatio > lfAspectRatio2)
        lfFactor = 1. * m_lfWindowWidth / image.cols;
    else
        lfFactor = 1. * m_lfWindowHeight / image.rows;
    double lfImgX = (m_lfWindowWidth / lfFactor - image.cols) / 2;
    double lfImgY = (m_lfWindowHeight / lfFactor - image.rows) / 2;

    emit sigImageUpdate(image.cols, image.rows, lfFactor, lfImgX, lfImgY);

    m_lfZoomFactor = lfFactor;
    m_lfImgX = lfImgX;
    m_lfImgY = lfImgY;
}


// main.cpp 代码如下：
#include <QGuiApplication>
#include <QQmlApplicationEngine>
#include <QQmlContext>
#include "drawimagectrl.h"

void initDrawImage(QQmlApplicationEngine &engine)
{
    engine.rootContext()->setContextProperty(QLatin1String("drawImageCtrl"), DrawImageCtrl::getInstance());
    engine.addImageProvider(QLatin1String("drawImg"), DrawImageCtrl::getInstance()->getImageProvider()); //添加图片提供者
}

int main(int argc, char *argv[])
{
#if QT_VERSION < QT_VERSION_CHECK(6, 0, 0)
    QCoreApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
#endif
    QGuiApplication app(argc, argv);

    QQmlApplicationEngine engine;
    const QUrl url(QStringLiteral("qrc:/main.qml"));
    QObject::connect(&engine, &QQmlApplicationEngine::objectCreated,
                     &app, [url](QObject *obj, const QUrl &objUrl) {
        if (!obj && url == objUrl)
            QCoreApplication::exit(-1);
    }, Qt::QueuedConnection);
    initDrawImage(engine);
    engine.load(url);

    return app.exec();
}


// main.qml 代码如下：
import QtQuick 2.15
import QtQuick.Window 2.15
import QtQuick.Controls 2.15

Window {
    width: 640
    height: 480
    visible: true
    title: qsTr("Hello World")

    DrawImage {
        anchors.fill: parent
    }

    Button {
        text: "draw"

        onClicked: {
            drawImageCtrl.sltTestDrawImage();
        }
    }
}


// DrawImage.qml 代码如下：
import QtQuick 2.15

Rectangle {
    id : canvasRect

    Connections {
        target: drawImageCtrl
        function onSigImageUpdate(lfImgWidth,lfImgHeight, lfFactor, lfImgX, lfImgY) {
            // 当 Canvas 宽度为 0 时，有严重的内存泄露.
            // 考虑到浮点数精度问题，直接小于1就不绘制图了.
            if(imgCanvas.width < 1 || imgCanvas.height < 1) {
                return;
            }

            if(0 !== imgCanvas.preImgUrl.length) {
                imgCanvas.unloadImage(imgCanvas.preImgUrl);
            }

            imgCanvas.imgRealWidth = lfImgWidth;
            imgCanvas.imgRealHeight = lfImgHeight;
            imgCanvas.factor = lfFactor;
            imgCanvas.imgX = lfImgX;
            imgCanvas.imgY = lfImgY;
            imgCanvas.imgUrl = "image://drawImg/"+Math.random()*100000;
            imgCanvas.loadImage(imgCanvas.imgUrl);
        }
    }

    Canvas {
        id : imgCanvas
        width: parent.width
        height: parent.height
        property var imgUrl: "image://drawImg/"
        property real imgRealWidth
        property real imgRealHeight
        property real factor
        property real imgX
        property real imgY
        property real preFactor: -1
        property string preImgUrl: ""
        property real preCanvasWidth: 0
        property real preCanvasHeight: 0

        onImageLoaded: {
            requestPaint();
        }

        onPaint: {
            let ctx = getContext("2d");

            if(0 !== imgCanvas.factor && imgCanvas.factor !== preFactor) {
                if(-1 !== preFactor) {
                    ctx.clearRect(0, 0, imgCanvas.preCanvasWidth, imgCanvas.preCanvasHeight);
                    ctx.scale(1/preFactor, 1/preFactor);
                }
                ctx.scale(imgCanvas.factor, imgCanvas.factor);
                preFactor = imgCanvas.factor;
            }

            ctx.clearRect(0, 0, imgCanvas.width, imgCanvas.height);
            ctx.drawImage(imgUrl,imgCanvas.imgX,imgCanvas.imgY,imgCanvas.imgRealWidth,imgCanvas.imgRealHeight);
            imgCanvas.preImgUrl = imgCanvas.imgUrl;
            imgCanvas.preCanvasWidth = imgCanvas.width;
            imgCanvas.preCanvasHeight = imgCanvas.height;
        }

        MouseArea {
            anchors.fill: parent
            hoverEnabled: true
            propagateComposedEvents: true
            acceptedButtons: Qt.LeftButton | Qt.RightButton | Qt.MidButton

            onEntered: {
                imgCanvas.forceActiveFocus();
            }

            onPressed: {
                drawImageCtrl.sltMousePressed(mouse.x, mouse.y, mouse.button);
            }

            onReleased: {
                drawImageCtrl.sltMouseReleased(mouse.x, mouse.y, mouse.button);
            }
        }

        Keys.onPressed: {
            drawImageCtrl.sltKeyPressed(event.key);
        }
        Keys.onReleased: {
            drawImageCtrl.sltKeyReleased(event.key);
        }

        onWidthChanged: {
            drawImageCtrl.sltWindowWidthChanged(imgCanvas.width);
        }

        onHeightChanged: {
            drawImageCtrl.sltWindowHeightChanged(imgCanvas.height);
        }
    }

    onVisibleChanged: {
        if(canvasRect.visible) {
            imgCanvas.focus = true;
        }
        else {
            imgCanvas.focus = false;
        }
    }

    onFocusChanged: {
        if(canvasRect.focus)
            imgCanvas.focus = true;
    }
}
```
