# sloth

## Intro

[![Build Status](https://travis-ci.org/cvhciKIT/sloth.svg)](https://travis-ci.org/cvhciKIT/sloth)

sloth is a tool for labeling image and video data for computer vision research.

The documentation can be found at http://sloth.readthedocs.org/ .

### Latest Releases

2013/11/29 v1.0: 2e69fdae40f89050fbaeef22491eee2a92e78b4f [.zip](https://github.com/cvhciKIT/sloth/archive/v1.0.zip) [.tar.gz](https://github.com/cvhciKIT/sloth/archive/v1.0.tar.gz)

For a full list, visit https://github.com/cvhciKIT/sloth/releases

# Installation
### Acer preinstallations - Ubuntu

After Installing Ubuntu on the laptop. Download rEFInd disc image, and create a bootable USB using Rufus
rEFInd: https://sourceforge.net/projects/refind/ 
Rufus: https://rufus.akeo.ie/ 

Enter your BIOS Boot settings. Under Boot Tab, make sure Secure Mode is disabled. Set your boot priority accordingly.
For Acer: Press F2 to enter BIOS Settings 

Upon restarting, you should be able to see the rEFInd boot loader screen.



Select rEFInd’s UEFI Shell, type the following:
```
bcfg boot dump
```
This prints a list of all available boot drivers. Ubuntu will not be here, yet.

bcfg boot add X fs0:\EFI\ubuntu\shimx64.efi "ubuntu"
Replace X with the next available number as shown on the list. It’s usually 3 for Acer Aspire Laptops.
If the EFI files are not in fs0, you can check each location by using fsN: & ls 
```
bcfg boot mv 3 0
```
This moves the ubuntu boot loader from 4th priority to 1st. This will set Ubuntu as the default OS.

Remove the USB and reboot

### sloth prerequisites (on Ubuntu)
1. pip
```
$ sudo apt-get update
$ sudo apt-get -y upgrade
$ sudo apt-get install -y python-pip
$ sudo pip install --upgrade pip
```
2. git
```
$ sudo apt-get install -y git
```
3. PyQt4
```
$ sudo apt-get install -y python-qt4
```
4. ffmpeg
```
$ sudo apt-get install -y ffmpeg
```
5. moviepy
```
$ sudo apt-get install -y moviepy
```
6. sloth
```
$ cd ~/Desktop
$ git clone https://github.com/mrnivlac/sloth.git
$ cd sloth
$ sudo python setup.py install
```
# Using sloth for video annotations
1. Create a file in the same directory as the video to be annotated and the corresponding json file.
```
$ touch sample.py
```
2. Open it and copy this into the file.
```
from PyQt4.QtGui import QPen
from PyQt4.Qt import Qt
from sloth.items import *
import math

class RectItem2(BaseItem):
    defaultAutoTextKeys = ['id','area']
    
    def __init__(self, model_item=None, prefix="", parent=None):
        BaseItem.__init__(self, model_item, prefix, parent)

        self._id = 1
	self._area = None
        self._rect = None
        self._resize = False
        self._resize_start = None
        self._resize_start_rect = None
        self._upper_half_clicked = None
        self._left_half_clicked = None

        self._updateID(self._dataToID(self._model_item))
	self._updateAREA(self._dataToAREA(self._model_item))
        self._updateRect(self._dataToRect(self._model_item))
        LOG.debug("Constructed rect %s for model item %s" %
                  (self._rect, model_item))

    def __call__(self, model_item=None, parent=None):
        item = RectItem2(model_item, parent)
        item.setPen(self.pen())
        item.setBrush(self.brush())
        return item

    def _dataToID(self, model_item):
        if model_item is None:
            return 1

        try:
            return model_item[self.prefix() + 'id']

        except KeyError as e:
            LOG.debug("RectItem2: Could not find expected key in item: "
                      + str(e) + ". Check your config!")
            self.setValid(False)
            return 1

    def _dataToAREA(self, model_item):
        if model_item is None:
            return 1

        try:
            return model_item[self.prefix() + 'area']

        except KeyError as e:
            LOG.debug("RectItem2: Could not find expected key in item: "
                      + str(e) + ". Check your config!")
            self.setValid(False)
            return 1	

    def _dataToRect(self, model_item):
        if model_item is None:
            return QRectF()

        try:
            return QRectF(int(model_item[self.prefix() + 'x']),
                          int(model_item[self.prefix() + 'y']),
                          int(model_item[self.prefix() + 'width']),
                          int(model_item[self.prefix() + 'height']))
        except KeyError as e:
            LOG.debug("RectItem2: Could not find expected key in item: "
                      + str(e) + ". Check your config!")
            self.setValid(False)
            return QRectF()

    def _updateID(self, ID):
        if ID == self._id:
            return

        self._id = ID

    def _updateAREA(self, AREA):
        if AREA == self._area:
            return

        self.area = AREA

    def _updateRect(self, rect):
        if rect == self._rect:
            return

        self.prepareGeometryChange()
        self._rect = rect
        self.setPos(rect.topLeft())

    def updateModel(self):
        self._rect = QRectF(self.scenePos(), self._rect.size())
        self._model_item.update({
            self.prefix() + 'x':      float(self._rect.topLeft().x()),
            self.prefix() + 'y':      float(self._rect.topLeft().y()),
            self.prefix() + 'width':  float(self._rect.width()),
            self.prefix() + 'height': float(self._rect.height()),	    
        })

    def boundingRect(self):
        return QRectF(QPointF(0, 0), self._rect.size())

    def paint(self, painter, option, widget=None):
        BaseItem.paint(self, painter, option, widget)

        pen = self.pen()
        if self.isSelected():
            pen.setStyle(Qt.DashLine)
        painter.setPen(pen)
        painter.drawRect(self.boundingRect())

    def dataChange(self):
        rect = self._dataToRect(self._model_item)
        self._updateRect(rect)

    def mousePressEvent(self, event):
        if event.button() & Qt.RightButton != 0:
            self._resize = True
            self._resize_start = event.scenePos()
            self._resize_start_rect = QRectF(self._rect)
            self._upper_half_clicked = (event.scenePos().y() < self._resize_start_rect.center().y())
            self._left_half_clicked  = (event.scenePos().x() < self._resize_start_rect.center().x())
            event.accept()
        else:
            BaseItem.mousePressEvent(self, event)

    def mouseMoveEvent(self, event):
        if self._resize:
            diff = event.scenePos() - self._resize_start
            if self._left_half_clicked:
                x = self._resize_start_rect.x() + math.floor(diff.x())
                w = self._resize_start_rect.width() - math.floor(diff.x())
            else:
                x = self._resize_start_rect.x()
                w = self._resize_start_rect.width() + math.floor(diff.x())

            if self._upper_half_clicked:
                y = self._resize_start_rect.y() + math.floor(diff.y())
                h = self._resize_start_rect.height() - math.floor(diff.y())
            else:
                y = self._resize_start_rect.y()
                h = self._resize_start_rect.height() + math.floor(diff.y())

            rect = QRectF(QPointF(x,y), QSizeF(w, h)).normalized()

            self._updateRect(rect)
            self.updateModel()
            event.accept()
        else:
            BaseItem.mouseMoveEvent(self, event)

    def mouseReleaseEvent(self, event):
        if self._resize:
            self._resize = False
            event.accept()
        else:
            BaseItem.mouseReleaseEvent(self, event)

    def keyPressEvent(self, event):
        BaseItem.keyPressEvent(self, event)
        step = 1
        if event.modifiers() & Qt.ShiftModifier:
            step = 5
        ds = {Qt.Key_Left:  (-step, 0),
              Qt.Key_Right: (step, 0),
              Qt.Key_Up:    (0, -step),
              Qt.Key_Down:  (0, step),
             }.get(event.key(), None)
        ds2 = {Qt.Key_Left:  (-1),
              Qt.Key_Right: (1),
              Qt.Key_Up:    (2),
              Qt.Key_Down:  (-2),
             }.get(event.key(), None)

        if ds is not None:
            if event.modifiers() & Qt.AltModifier:
                ID = self._id + ds2
                self._updateID(ID)
            else:
                if event.modifiers() & Qt.ControlModifier:
                    rect = self._rect.adjusted(*((0, 0) + ds))
                else:
                    rect = self._rect.adjusted(*(ds + ds))
                self._updateRect(rect)
            self.updateModel()
            event.accept()

	if ds is not None:
            if event.modifiers() & Qt.AltModifier:
                AREA = self._area + ds2
                self._updateAREA(AREA)
            else:
                if event.modifiers() & Qt.ControlModifier:
                    rect = self._rect.adjusted(*((0, 0) + ds))
                else:
                    rect = self._rect.adjusted(*(ds + ds))
                self._updateRect(rect)
            self.updateModel()
            event.accept()

LABELS = (
    {
        'attributes': {
            'class': 'rect',
	    'area': ['1','2','3','4','5'],
            'id':   ['1','2','3','4','5']
	    
        },
        'inserter':  'sloth.items.RectItemInserter',
        'item':     RectItem2,
        'hotkey':   'r',
        'text':     'Rectangle',
    },
)

HOTKEYS = (
    ('Space',     [lambda lt: lt.currentImage().confirmAll(),
                   lambda lt: lt.currentImage().setUnlabeled(False),
                   lambda lt: lt.gotoNext()
                  ],                                         'Mark image as labeled/confirmed and go to next'),
    ('Backspace', lambda lt: lt.gotoPrevious(),              'Previous image/frame'),
    ('PgDown',    lambda lt: lt.gotoNext(),                  'Next image/frame'),
    ('PgUp',      lambda lt: lt.gotoPrevious(),              'Previous image/frame'),
    ('Tab',       lambda lt: lt.selectNextAnnotation(),      'Select next annotation'),
    ('Shift+Tab', lambda lt: lt.selectPreviousAnnotation(),  'Select previous annotation'),
    ('Ctrl+f',    lambda lt: lt.view().fitInView(),          'Fit current image/frame into window'),
    ('Del',       lambda lt: lt.deleteSelectedAnnotations(), 'Delete selected annotations'),
    ('ESC',       lambda lt: lt.exitInsertMode(),            'Exit insert mode'),
    ('Shift+l',   lambda lt: lt.currentImage().setUnlabeled(False), 'Mark current image as labeled'),
    ('Shift+c',   lambda lt: lt.currentImage().confirmAll(), 'Mark all annotations in image as confirmed'),
)

CONTAINERS = (
    ('*.json',       'sloth.annotations.container.JsonContainer'),
)
```
* To annotate more than one label, edit the variable named LABELS and add a key under the 'attributes' key named 'type'.
