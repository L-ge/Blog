1. 大体功能：
- 频繁地往界面推送图片，帧率达到视频效果。
- 捕获画布上的鼠标事件和键盘事件。

2. 代码如下：
```cpp
// DrawImageInQQuickPaintedItem.pro 代码如下：
QT += quick

# You can make your code fail to compile if it uses deprecated APIs.
# In order to do so, uncomment the following line.
#DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    # disables all the APIs deprecated before Qt 6.0.0

SOURCES += \
        drawimagectrl.cpp \
        imagepainter.cpp \
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
    imagepainter.h

LIBS += -lopencv_core \
        -lopencv_imgcodecs


// imagepainter.h 代码如下：
#ifndef IMAGEPAINTER_H
#define IMAGEPAINTER_H

#include <QQuickPaintedItem>
#include <QImage>

#define QML_WRITABLE_PROPERTY(type, name) \
    protected: \
        Q_PROPERTY (type name READ get_##name WRITE set_##name NOTIFY name##Changed) \
        type m_##name; \
    public: \
        type get_##name () const { \
            return m_##name ; \
        } \
    public Q_SLOTS: \
        bool set_##name (type name) { \
            bool ret = false; \
            if ((ret = m_##name != name)) { \
                m_##name = name; \
                emit name##Changed (m_##name); \
            } \
            return ret; \
        } \
    Q_SIGNALS: \
        void name##Changed (type name); \
    private:

class MyQImage : public QObject
{
    Q_OBJECT
public:
    explicit MyQImage(QObject *parent = nullptr) : QObject(parent) {}
    ~MyQImage() = default;

    void setImage(const QImage &img) { m_img = img; }
    const QImage &getImage() const { return m_img; }

private:
    QImage m_img;
};

class ImagePainter : public QQuickPaintedItem
{
    Q_OBJECT
    QML_WRITABLE_PROPERTY(MyQImage*, pImage)
    QML_WRITABLE_PROPERTY(int, nFillMode)

public:
    explicit ImagePainter(QQuickItem *parent = nullptr);

    void paint(QPainter *painter) override;
};

#endif // IMAGEPAINTER_H


// imagepainter.cpp 代码如下：
#include "imagepainter.h"
#include <QPainter>

ImagePainter::ImagePainter(QQuickItem *parent)
    : QQuickPaintedItem(parent)
    , m_pImage(nullptr)
    , m_nFillMode(0)
{
    connect(this, &ImagePainter::pImageChanged,
            [this]{
        if(m_pImage) {
            setImplicitWidth(m_pImage->getImage().width());
            setImplicitHeight(m_pImage->getImage().height());
        }
        else {
            setImplicitWidth(0);
            setImplicitHeight(0);
        }

        update();
    });
}

void ImagePainter::paint(QPainter *painter)
{
    if(m_pImage) {
        const QImage &image = m_pImage->getImage();
        if (image.isNull()) {
            return;
        }
        switch (m_nFillMode) {
        case 1 /* PreserveAspectFit */: {
            QImage scaledImage = image.scaled(width(), height(), Qt::KeepAspectRatio);
            double x = (width() - scaledImage.width()) / 2;
            double y = (height() - scaledImage.height()) / 2;
            painter->drawImage(QPoint(x, y), scaledImage);
            break;
        }
        case 0 /* Stretch */:
        default: {
            painter->drawImage(QPoint(0, 0), image.scaled(width(), height()));
        }
        }
    }
}


// drawimagectrl.h 代码如下：
#ifndef DRAWIMAGECTRL_H
#define DRAWIMAGECTRL_H

#include <QObject>
#include "opencv2/opencv.hpp"
#include "imagepainter.h"

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

#define QML_CONSTANT_PROPERTY(type, name) \
    protected: \
        Q_PROPERTY (type name READ get_##name CONSTANT) \
        type m_##name; \
    public: \
        type get_##name () const { \
            return m_##name ; \
        } \
    private:

class DrawImageCtrl : public QObject
{
    Q_OBJECT
    SINGLETON(DrawImageCtrl)
    QML_CONSTANT_PROPERTY(MyQImage*, pSceneImage)

public:
    explicit DrawImageCtrl(QObject *parent = nullptr);
    ~DrawImageCtrl();

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
    void sigImageUpdate();

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
};

#endif // DRAWIMAGECTRL_H


// drawimagectrl.cpp 代码如下：
#include "drawimagectrl.h"
#include <QTimer>
#include <QDebug>

DrawImageCtrl::DrawImageCtrl(QObject *parent)
    : QObject(parent)
    , m_pSceneImage(new MyQImage(this))
    , m_lfWindowWidth(0.0)
    , m_lfWindowHeight(0.0)
    , m_lfLastMouseX(-1)
    , m_lfLastMouseY(-1)
    , m_lfZoomFactor(1.0)
    , m_lfImgX(0.0)
    , m_lfImgY(0.0)
{

}

DrawImageCtrl::~DrawImageCtrl()
{
    if(nullptr != m_pSceneImage)
    {
        delete m_pSceneImage;
        m_pSceneImage = nullptr;
    }
}

void DrawImageCtrl::sltTestDrawImage()
{
    static int i = 0;
    QString sPicFileName = QString::fromUtf8("/home/xiaohuamao/mycode/DrawImageInQQuickPaintedItem/TestPic/%1.png").arg(i);
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

QImage cvMatToQImage( const cv::Mat &inMat )
{
   switch ( inMat.type() )
   {
      // 8-bit, 4 channel
      case CV_8UC4:
      {
         QImage image( inMat.data,
                       inMat.cols, inMat.rows,
                       static_cast<int>(inMat.step),
                       QImage::Format_ARGB32 );

         return image;
      }

      // 8-bit, 3 channel
      case CV_8UC3:
      {
         QImage image( inMat.data,
                       inMat.cols, inMat.rows,
                       static_cast<int>(inMat.step),
                       QImage::Format_RGB888 );

         return image.rgbSwapped();
      }

      // 8-bit, 1 channel
      case CV_8UC1:
      {
#if QT_VERSION >= QT_VERSION_CHECK(5, 5, 0)
         QImage image( inMat.data,
                       inMat.cols, inMat.rows,
                       static_cast<int>(inMat.step),
                       QImage::Format_Grayscale8 );
#else
         static QVector<QRgb>  sColorTable;

         // only create our color table the first time
         if ( sColorTable.isEmpty() )
         {
            sColorTable.resize( 256 );

            for ( int i = 0; i < 256; ++i )
            {
               sColorTable[i] = qRgb( i, i, i );
            }
         }

         QImage image( inMat.data,
                       inMat.cols, inMat.rows,
                       static_cast<int>(inMat.step),
                       QImage::Format_Indexed8 );

         image.setColorTable( sColorTable );
#endif

         return image;
      }

      default:
         qWarning() << "cvMatToQImage() - cv::Mat image type not handled in switch:" << inMat.type();
         break;
   }

   return QImage();
}

void DrawImageCtrl::showImage(const cv::Mat &mat)
{
    cv::Mat image = mat.clone();
    m_pSceneImage->setImage(cvMatToQImage(mat));

    auto lfAspectRatio = 1. * image.cols / image.rows;
    auto lfAspectRatio2 = 1. * m_lfWindowWidth / m_lfWindowHeight;
    auto lfFactor = 1.0;
    if (lfAspectRatio > lfAspectRatio2)
        lfFactor = 1. * m_lfWindowWidth / image.cols;
    else
        lfFactor = 1. * m_lfWindowHeight / image.rows;
    double lfImgX = (m_lfWindowWidth / lfFactor - image.cols) / 2;
    double lfImgY = (m_lfWindowHeight / lfFactor - image.rows) / 2;

    emit sigImageUpdate();

    m_lfZoomFactor = lfFactor;
    m_lfImgX = lfImgX;
    m_lfImgY = lfImgY;
}


// main.cpp 代码如下：
#include <QGuiApplication>
#include <QQmlApplicationEngine>
#include <QQmlContext>
#include "drawimagectrl.h"
#include "imagepainter.h"

void initDrawImage(QQmlApplicationEngine &engine)
{
    qmlRegisterType<ImagePainter>("MyItem", 1, 0, "ImagePainter");
    engine.rootContext()->setContextProperty(QLatin1String("drawImageCtrl"), DrawImageCtrl::getInstance());
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
import QtQuick 2.0
import MyItem 1.0

Rectangle {
    id : canvasRect

    Connections{
        target: drawImageCtrl

        function onSigImageUpdate(){
            idSceneImage.update();
        }
    }

    Image {
        id: imgCanvas
        width: parent.width
        height: parent.height
        anchors.fill: parent

        ImagePainter {
            id: idSceneImage
            anchors.fill: parent
            nFillMode: Image.PreserveAspectFit
            pImage: drawImageCtrl.pSceneImage
            visible: true
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
