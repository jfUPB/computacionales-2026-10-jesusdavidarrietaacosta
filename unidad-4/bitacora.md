# Unidad 4

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
main.cpp

```
#include "ofMain.h"
#include "ofApp.h"

int main() {
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new ofApp());
    return 0;
}

```

ofApp.h

```
#pragma once
#include "ofMain.h"

// Nodo de la cola
struct Node {
    float x, y;
    float radius;
    ofColor color;
    float opacity;
    Node* next;
    Node(float _x, float _y, float _radius, ofColor _color, float _opacity)
        : x(_x), y(_y), radius(_radius), color(_color), opacity(_opacity), next(nullptr) {}
};

// Implementación manual de una cola (FIFO)
class BrushQueue {
public:
    Node* front;
    Node* rear;
    int size;
    int maxSize;

    BrushQueue(int _maxSize);
    ~BrushQueue();
    void enqueue(float x, float y, float radius, ofColor color, float opacity);
    void dequeue();
    void clear();
    bool isEmpty();
};

// Constructor
BrushQueue::BrushQueue(int _maxSize)
    : front(nullptr), rear(nullptr), size(0), maxSize(_maxSize) {}

// Destructor
BrushQueue::~BrushQueue() {
    clear();
}

// enqueue: agregar nodo al final
void BrushQueue::enqueue(float x, float y, float radius, ofColor color, float opacity) {
    Node* newNode = new Node(x, y, radius, color, opacity);
    if (isEmpty()) {
        front = rear = newNode;
    } else {
        rear->next = newNode;
        rear = newNode;
    }
    size++;
    if (size > maxSize) {
        dequeue();
    }
}

// dequeue: eliminar el nodo más antiguo
void BrushQueue::dequeue() {
    if (isEmpty()) return;
    Node* temp = front;
    front = front->next;
    if (front == nullptr) rear = nullptr;
    delete temp;
    size--;
}

// clear: eliminar todos los nodos
void BrushQueue::clear() {
    while (!isEmpty()) {
        dequeue();
    }
}

// isEmpty: verificar si la cola está vacía
bool BrushQueue::isEmpty() {
    return front == nullptr;
}

// ofApp: strokes es miembro de la clase, no global
class ofApp : public ofBaseApp {
public:
    BrushQueue strokes;
    float backgroundHue = 0;

    ofApp() : strokes(50) {}

    void setup();
    void update();
    void draw();
    void mouseMoved(int x, int y);
    void keyPressed(int key);
    void keyReleased(int key);
    void mouseDragged(int x, int y, int button);
    void mousePressed(int x, int y, int button);
    void mouseReleased(int x, int y, int button);
    void mouseEntered(int x, int y);
    void mouseExited(int x, int y);
    void windowResized(int w, int h);
    void dragEvent(ofDragInfo dragInfo);
    void gotMessage(ofMessage msg);
};

```

ofapp.cpp

```
#include "ofApp.h"

//--------------------------------------------------------------
void ofApp::setup() {
    ofBackground(0);
}

//--------------------------------------------------------------
void ofApp::update() {
    backgroundHue += 0.2;
    if (backgroundHue > 255) backgroundHue = 0;
}

//--------------------------------------------------------------
void ofApp::draw() {
    // Fondo con gradiente dinámico
    ofColor color1, color2;
    color1.setHsb(backgroundHue, 150, 240);
    color2.setHsb(fmod(backgroundHue + 128, 255), 150, 240);
    ofBackgroundGradient(color1, color2, OF_GRADIENT_LINEAR);

    // Recorrer y dibujar los trazos con fade por antigüedad
    int index = 0;
    Node* current = strokes.front;
    while (current != nullptr) {
        float ageFade = ofMap(index, 0, strokes.size - 1, 60, 255);
        ofSetColor(current->color, ageFade);
        ofDrawCircle(current->x, current->y, current->radius);
        current = current->next;
        index++;
    }
}

//--------------------------------------------------------------
// Al mover el mouse se agrega un trazo con color aleatorio
void ofApp::mouseMoved(int x, int y) {
    float radius = ofRandom(10, 20);
    ofColor color;
    color.setHsb(ofRandom(255), 255, 255);
    float opacity = 255;
    strokes.enqueue(x, y, radius, color, opacity);
}



//--------------------------------------------------------------
void ofApp::keyPressed(int key) {
    if (key == 'c') {
        strokes.clear();
    }
    if (key == 'a') {
        strokes.maxSize = (strokes.maxSize == 50) ? 100 : 50;
        // Recortar nodos sobrantes si se baja de 100 a 50
        while (strokes.size > strokes.maxSize) {
            strokes.dequeue();
        }
    } else if (key == 's') {
        ofSaveFrame();
    }
}

// Stubs requeridos por ofBaseApp
void ofApp::keyReleased(int key) {}
void ofApp::mouseDragged(int x, int y, int button) {}
void ofApp::mousePressed(int x, int y, int button) {}
void ofApp::mouseReleased(int x, int y, int button) {}
void ofApp::mouseEntered(int x, int y) {}
void ofApp::mouseExited(int x, int y) {}
void ofApp::windowResized(int w, int h) {}
void ofApp::dragEvent(ofDragInfo dragInfo) {}
void ofApp::gotMessage(ofMessage msg) {}
```
<img width="1674" height="971" alt="Captura de pantalla 2026-04-17 065130" src="https://github.com/user-attachments/assets/2d13972f-ed8c-4da4-a4d1-5043ba5038b8" />

videncia 1:La inserción del primer nodo en una cola vacía ocurre en la linea de codigo 
if (isEmpty()) {
    front = rear = newNode;
}
específicamente en la línea front = rear = newNode;, donde ambos punteros pasan a referenciar el nuevo nodo creado gracias a que isEmpty() verifica si front == nullptr, es decir, la cola está vacía y cuando esto pasa ese nuevo nodo pasa a ser el primero y el ultimo.

Evidencia 2: El mantenimiento del orden FIFO ocurre en las líneas rear->next = newNode; y rear = newNode;, ya que los nuevos elementos siempre se insertan al final de la cola sin alterar el orden de los elementos existentes.lo primero que entra lo primero que sale osea first in first on
<img width="1675" height="941" alt="Captura de pantalla 2026-04-17 070002" src="https://github.com/user-attachments/assets/2d2ea329-cd76-4df8-b8bd-863a51e2df96" />

Evidencia 3: La eliminación en la cola se realiza desde el puntero front, manteniendo el orden FIFO. Para evitar fugas de memoria, el nodo eliminado se libera explícitamente con delete temp, ya que fue reservado dinámicamente con new.

<img width="1674" height="908" alt="image" src="https://github.com/user-attachments/assets/dd446355-4288-49c3-8687-03d774c5bc6a" />

Evidencia 4: El control del tamaño máximo se implementa incrementando el contador size tras cada inserción y verificando si excede maxSize. En ese caso, se elimina el nodo más antiguo mediante dequeue(), garantizando que la cola no supere el tamaño permitido y conserve únicamente los elementos más recientes.

<img width="1676" height="942" alt="Captura de pantalla 2026-04-17 071922" src="https://github.com/user-attachments/assets/60ab1f4f-7c66-4353-bb3e-c109645f2094" />

Evidencia 5: Se usa un puntero auxiliar current para recorrer la cola desde front hasta nullptr, sin modificar ni eliminar ningún nodo.

<img width="1678" height="907" alt="Captura de pantalla 2026-04-17 072635" src="https://github.com/user-attachments/assets/c0359731-57c1-4e79-ba48-5ff8e370af97" />

Evidencia 6: Llama a dequeue() repetidamente hasta que la cola esté vacía. Como dequeue() hace delete en cada nodo, todos los nodos son liberados correctamente.

<img width="1676" height="907" alt="Captura de pantalla 2026-04-17 072928" src="https://github.com/user-attachments/assets/13641280-da05-43ec-9c5d-c0ba85976145" />


## Bitácora de reflexión
