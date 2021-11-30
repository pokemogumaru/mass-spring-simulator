## Welcome to my mass-spring simulator
Please click on the link on the top right (https://github.com/pokemogumaru/mass-spring-simulator) to access the github repo.

![image](https://user-images.githubusercontent.com/72077449/144141776-db0aa123-78fa-4273-8af7-f1d17049bfb3.png)

Code for version 18_1:
(Indentation is invalid due to github and you need PYQT5)
import sys, math
from PyQt5.QtGui import *
from PyQt5.QtCore import *
from PyQt5.QtWidgets import *

k = 1.0
MOVE_MASS = 0
ADD_SPRING = 1

class spring:
    def __init__(self, A, B, k):
        self.A = A #Anchor
        self.B = B #Mass
        self.k = k #spring constant
        dx = self.B.x_position - self.A.x_position 
        dy = self.B.y_position - self.A.y_position
        self.rest_length = math.sqrt(dx*dx + dy*dy)

   def force(self):#, x_position, y_position):
        #length = math.sqrt((x_position * x_position)+(y_position * y_position))
        dx = self.B.x_position - self.A.x_position
        dy = self.B.y_position - self.A.y_position
        length = math.sqrt(dx*dx + dy*dy)
        #print('length', length)
        force_magnitude = self.k * (length - self.rest_length)
        #print('fmag', force_magnitude)
        x_direction = dx/length
        y_direction = dy/length
        #print(x_direction, y_direction)
        self.A.x_force = self.A.x_force + force_magnitude * x_direction
        self.A.y_force = self.A.y_force + force_magnitude * y_direction
        self.B.x_force = self.B.x_force - force_magnitude * x_direction
        self.B.y_force = self.B.y_force - force_magnitude * y_direction
        return [(force_magnitude * x_direction), (force_magnitude * y_direction)]

   def x_force(self, length):
        return -self.k * (length - self.rest_length)

   def y_force(self, length):
        return -self.k * (length - self.rest_length)

class block:
    def __init__(self, mass, x_position, y_position, user_damping, dynamic):
        self.mass = mass
        self.x_position = x_position
        self.y_position = y_position
        self.x_velocity = 0
        self.y_velocity = 0
        self.x_acceleration = 0
        self.y_acceleration = 0
        self.x_force = 0
        self.y_force = 0
        self.dynamic = dynamic
        self.user_damping = user_damping

   def frame(self, time):#, x_force, y_force):
        if self.dynamic == True :
            self.x_acceleration = self.x_force / self.mass
            self.x_velocity += self.x_acceleration * time
            self.x_position += self.x_velocity * time
            self.y_acceleration = self.y_force / self.mass
            self.y_velocity += self.y_acceleration * time
            self.y_position += self.y_velocity * time

   def damp(self):
        if self.dynamic == True :
            self.x_velocity = self.x_velocity * self.user_damping
            self.y_velocity = self.y_velocity * self.user_damping

#Previous inputs/variables have changed

class canvas(QWidget):
    def __init__(self, main, width, height):
        super(canvas, self).__init__()
        self.main = main
        self.setMinimumSize(width,height)
        self.width = width
        self.height = height
        self.mouse_x = 0
        self.mouse_y = 0
        self.dragging = -1
        self.state = MOVE_MASS
        self.spring = []
        self.setMouseTracking(True)

   def mouseMoveEvent(self, event):
        #print('Mouse coords: ( %d : %d )' % (event.x(), event.y()))
        self.mouse_x = event.x()
        self.mouse_y = event.y()

   if self.dragging >=0 :
            #print(self.dragging)
            m = self.main.masses[self.dragging]
            m[0] = self.mouse_x
            m[1] = self.mouse_y
            self.repaint()

   def mousePressEvent(self, event):
        #print('button pressed')
        min_distance = 1000
        closest = -1
        
   #for m in masses:
        for i in range(len(self.main.masses)):
            m = self.main.masses[i]
            distance = math.sqrt((m[0]-self.mouse_x)*(m[0]-self.mouse_x)+(m[1]-self.mouse_y)*(m[1]-self.mouse_y))
            #print(i, distance)
            if distance <= 14.4 and distance < min_distance:
                distance = min_distance
                closest = i

   if closest >= 0:
            if self.state == MOVE_MASS:
                self.dragging = closest
            elif self.state == ADD_SPRING:
                self.spring.append(closest)
                if len(self.spring) == 2:
                    self.spring.append(k)
                    self.main.springs.append(self.spring)
                    self.spring = []
                    self.state = MOVE_MASS
                    self.repaint()
                
   def mouseReleaseEvent(self, event):
        #print('button release')
        self.dragging = -1

   def paintEvent(self, event):
        #print('paint')
        qp = QPainter()
        qp.begin(self)

   if self.main.running == True:
            for b in self.main.blocks:
                #print(m)
                if b.dynamic==True:
                    qp.setPen(QColor(Qt.green))
                else:
                    qp.setPen(QColor(Qt.red))

   qp.drawRect(b.x_position - 10, b.y_position - 10, 20 , 20)
                qp.drawPoint(b.x_position, b.y_position)

   qp.setPen(QColor(Qt.blue))

  for s in self.main.blocks_springs:
                #m0 = masses[s[0]]
                #m1 = masses[s[1]]
                qp.drawLine(s.A.x_position,s.A.y_position, s.B.x_position, s.B.y_position)
        else:
            for m in self.main.masses:
                #print(m)
                if m[2]==True:
                    qp.setPen(QColor(Qt.green))
                else:
                    qp.setPen(QColor(Qt.red))

   qp.drawRect(m[0] - 10, m[1] - 10, 20 , 20)
                qp.drawPoint(m[0], m[1])
                
   qp.setPen(QColor(Qt.blue))

   for s in self.main.springs:
                m0 = self.main.masses[s[0]]
                m1 = self.main.masses[s[1]]
                qp.drawLine(m0[0],m0[1], m1[0], m1[1])
        
   qp.end()

class mywindow(QWidget):

  def tick(self):
        if self.running == True and self.paused == False:
            for b in self.blocks: # reset all forces
                b.x_force = 0
                b.y_force = 9.8 * b.mass

   for s in self.blocks_springs: # calculate force due to springs
                s.force()

   for b in self.blocks: # update block positions
                b.frame(0.1)

   self.canvas.repaint()

 def run(self):
        if not(self.running):
            self.paused = False
            
   self.blocks = []
            for m in self.masses:
                damp = float(self.damp_edit.text())
                self.blocks.append(block(m[3], m[0], m[1], damp, m[2]))

   self.blocks_springs = []
            for s in self.springs:
                self.blocks_springs.append(spring(self.blocks[s[0]], self.blocks[s[1]], s[2]))

  self.running = not(self.running)

   def pause(self):
        self.paused = not (self.paused)

  def dampingChanged(self,i):
        if i == 0:
            self.damp_edit.setEnabled(True)
        else:
            self.damp_edit.setEnabled(False)

  def remove(self):
        buttonReply = QMessageBox.question(self, 'Remove All?',  "This will remove everything, Are you sure?", QMessageBox.Yes | QMessageBox.No, QMessageBox.No)
        if buttonReply == QMessageBox.Yes:
            self.running = False
            self.paused = False
            self.blocks = []
            self.masses = []
            self.blocks_springs = []
            self.springs = []

  #self.masses = [] # [x, y, dynamic(boolean), mass]
    #self.springs = [] # [index 1, index 2, spring constant]

  def save(self):
        options = QFileDialog.Options()
        options |= QFileDialog.DontUseNativeDialog
        fileName, _ = QFileDialog.getSaveFileName(self,"save simulation","","Simulation Files (*.sim)", options=options)
        if fileName:
            if not(fileName.endswith(".sim")):
                fileName = fileName + ".sim"

   print(fileName)

   f = open(fileName, "w")
            f.write(str(len(self.masses))+"\n")
            for m in self.masses:
                f.write(str(m[0]) + "," + str(m[1]) + "," + str(m[2]) + "," + str(m[3]) + "\n")
            f.write(str(len(self.springs))+"\n")
            for s in self.springs:
                f.write(str(s[0]) + "," + str(s[1]) + "," + str(s[2]) + "\n")
            f.close()

  def load(self):
        options = QFileDialog.Options()
        options |= QFileDialog.DontUseNativeDialog
        fileName, _ = QFileDialog.getOpenFileName(self,"load simulation", "","Simulation Files (*.sim)", options=options)
        if fileName:
            print(fileName)
            f = open(fileName, "r")
            lines = []
            for line in f:
                lines.append(line.strip())
            print(lines)

   nmass = int(lines[0])
            nspring = int(lines[1 + nmass])
            masses = lines[1:(1+nmass)]
            springs = lines[(2+nmass):]

   self.running = False
            self.paused = False
            self.blocks = []
            self.masses = []
            self.blocks_springs = []
            self.springs = []

   for m in masses:
                data = m.split(",")
                print(data)
                self.masses.append([float(data[0]),float(data[1]),data[2] == "True",float(data[3])])

   for s in springs:
                data = s.split(",")
                print(data)
                self.springs.append([int(data[0]),int(data[1]),float(data[2])])

   print(self.masses)
            print(self.springs)
        
 def add_mass(self):
        self.announcement.setText("Add mass")
        self.masses.append([250,250,True,float(self.mass_edit.text())])

  def add_anchor(self):
        self.announcement.setText("Add anchor")
        self.masses.append([250,250,False,0.0])

  def add_spring(self):
        self.announcement.setText("Select two masses")
        #k = 
        self.canvas.state = ADD_SPRING
    
  def __init__(self):
        super(mywindow, self).__init__()

   self.masses = [] # [x, y, dynamic(boolean), mass]
        self.springs = [] # [index 1, index 2, spring constant]

   self.running = False
        self.paused = False
        self.blocks =  [] #block for simulation
        self.blocks_springs = [] #Storing instances of spring class
        
   self.setWindowTitle('Mass Spring Simulator')
        layout = QGridLayout()

   self.canvas = canvas(self, 500, 500)
        #self.canvas.setMinimumSize(500,500)
        self.canvas.show()
        layout.addWidget(self.canvas, 0, 1, 1, 2)

   panel = QWidget()
        layout2 = QGridLayout()

   button = QPushButton("Start/Restart")
        button.clicked.connect(self.run)
        layout2.addWidget(button, 1,1,1,1)        

   button_2 = QPushButton("Pause")
        button_2.clicked.connect(self.pause)
        layout2.addWidget(button_2, 1,2,1,1)

   panel.setLayout(layout2)
        layout.addWidget(panel, 1,1,1,2)

   button_3 = QPushButton("Remove All")
        button_3.clicked.connect(self.remove)
        layout.addWidget(button_3, 2,1,1,2)

   self.damp_cb = QComboBox()
        self.damp_cb.addItems(["Damping", "No Damping"])
        self.damp_cb.currentIndexChanged.connect(self.dampingChanged)
        layout.addWidget(self.damp_cb, 3,1,1,1)
        self.damp_edit = QLineEdit()
        self.damp_edit.setValidator(QDoubleValidator(0.0,1.0,3,self.damp_edit))
        self.damp_edit.setText('1.0')
        layout.addWidget(self.damp_edit,3,2,1,1)

   self.mass_label = QLabel("Mass")
        layout.addWidget(self.mass_label,4,1,1,1)
        self.mass_edit = QLineEdit()
        self.mass_edit.setValidator(QDoubleValidator(0.0,1.0,3,self.mass_edit))
        self.mass_edit.setText('1.0')
        layout.addWidget(self.mass_edit,4,2,1,1)

   self.mass_add = QPushButton("Add mass")
        self.mass_add.clicked.connect(self.add_mass)
        layout.addWidget(self.mass_add, 5,1,1,2)

   self.anchor_add = QPushButton("Add anchor")
        self.anchor_add.clicked.connect(self.add_anchor)
        layout.addWidget(self.anchor_add, 6,1,1,2)
        
   self.spring_add = QPushButton("Add Spring")
        self.spring_add.clicked.connect(self.add_spring)
        layout.addWidget(self.spring_add, 7,1,1,2)

  panel_3 = QWidget()
        layout3 = QGridLayout()

        self.save_button = QPushButton("Save Simulation")
   self.save_button.clicked.connect(self.save)
        layout3.addWidget(self.save_button,1,1,1,1)

   self.load_button = QPushButton("Load Simulation")
        self.load_button.clicked.connect(self.load)
        layout3.addWidget(self.load_button,1,2,1,1)

   panel_3.setLayout(layout3)
        layout.addWidget(panel_3, 8,1,1,2)

   self.announcement = QLabel()
        layout.addWidget(self.announcement, 9,1,1,1)

   #self.y_cb = QComboBox()
        #self.y_cb.addItems(["Y-centred", "Y-offset"])
        #layout.addWidget(self.y_cb,4,1,1,1)
        #self.y_edit = QLineEdit()
        #self.y_edit.setValidator(QDoubleValidator(0.0,1.0,3,self.y_edit))
        #layout.addWidget(self.y_edit,4,2,1,1)

   self.setLayout(layout)

   self.timer = QTimer()
        self.timer.timeout.connect(self.tick)
        self.timer.start(15)

app = QApplication(sys.argv)
window = mywindow()
window.show()

#qp = QPainter()
#qp.begin(window)
#qp.setPen(QColor(Qt.red))
#qp.drawLine(0,0,500,500)
#qp.end()

app.exec_()

```
You can use the [editor on GitHub](https://github.com/pokemogumaru/mass-spring-simulator/edit/master/docs/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown


```markdown
```Syntax highlighted code block

//# Header 1
//## Header 2
//### Header 3

//- Bulleted
//- List

//1. Numbered
//2. List

//**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

```For more details see [Basic writing and formatting syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/pokemogumaru/mass-spring-simulator/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.
