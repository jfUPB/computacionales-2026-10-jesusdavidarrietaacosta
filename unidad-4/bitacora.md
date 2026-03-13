# Unidad 4

## Bitácora de proceso de aprendizaje


## Bitácora de aplicación 
main.cpp

#include "ofMain.h"
#include "ofApp.h"

int main() {
    ofSetupOpenGL(1024, 768, OF_WINDOW);
    ofRunApp(new ofApp());
    return 0;
}

ofApp.h

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


ofapp.cpp


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


<img width="1904" height="1077" alt="image" src="https://github.com/user-attachments/assets/9a192ae5-2d71-47e1-a4fc-ec6f45480b60" />
<img width="1912" height="1143" alt="Captura de pantalla 2026-03-13 123809" src="https://github.com/user-attachments/assets/219b98ca-c083-4f56-a17d-cecce27ffdbf" />
<img width="1919" height="1151" alt="Captura de pantalla 2026-03-13 123820" src="https://github.com/user-attachments/assets/9e126f17-f226-48c5-ae06-f3ea666ac689" />
<img width="1911" height="1143" alt="Captura de pantalla 2026-03-13 123833" src="https://github.com/user-attachments/assets/f16874f9-04ff-4c4a-a008-ba308609e679" />
<img width="1919" height="1145" alt="Captura de pantalla 2026-03-13 123854" src="https://github.com/user-attachments/assets/d4cb8bc3-eade-41a6-ac51-bc36d3cfbb23" />
<img width="1919" height="1149" alt="Captura de pantalla 2026-03-13 123905" src="https://github.com/user-attachments/assets/9a25cc4f-1086-48d3-956c-c23106f148c3" />
<img width="1919" height="1141" alt="Captura de pantalla 2026-03-13 123916" src="https://github.com/user-attachments/assets/e4aa9920-f3b1-4182-a4b5-4223a07e6d75" />
<img width="1919" height="1139" alt="Captura de pantalla 2026-03-13 123933" src="https://github.com/user-attachments/assets/69caf2ac-63ec-459c-ac53-39a30cb61b18" />
<img width="1919" height="1140" alt="Captura de pantalla 2026-03-13 123756" src="https://github.com/user-attachments/assets/9bef5944-0ecd-4eeb-80cb-a81986fcd2c0" />

<img width="1672" height="209" alt="Captura de pantalla 2026-03-13 124059" src="https://github.com/user-attachments/assets/4f6f5e39-29bd-4d60-a764-0d0146b67265" />

videncia 1: Inserción del primer nodo en una cola vacía (enqueue)
if (isEmpty()) {
    front = rear = newNode;
}
Cuando la cola está vacía, front y rear son nullptr. Al insertar el primer nodo, ambos apuntan al mismo nodo. Esto demuestra que la cola puede inicializarse correctamente desde cero, cumpliendo el requisito de implementar la estructura desde cero sin librerías externas.

Evidencia 2: Mantenimiento del orden FIFO al insertar más nodos (enqueue)
} else {
    rear->next = newNode;
    rear = newNode;
}
Los nuevos nodos siempre se agregan al final (rear), y los más antiguos permanecen al frente (front). Esto garantiza el orden FIFO: el primero en entrar es el primero en salir, exactamente como lo pide el enunciado.

Evidencia 3: Comportamiento de eliminación y prevención de fugas (dequeue)
void BrushQueue::dequeue() {
    if (isEmpty()) return;
    Node* temp = front;
    front = front->next;
    if (front == nullptr) rear = nullptr;
    delete temp;
    size--;
}
Se guarda el puntero al nodo más antiguo en temp, se avanza front al siguiente, y luego se hace delete temp. Esto libera la memoria correctamente sin dejar fugas. Si la cola queda vacía, rear también se pone en nullptr para mantener consistencia.

Evidencia 4: Control del tamaño máximo de la cola (maxSize)
size++;
if (size > maxSize) {
    dequeue();
}
Justo después de insertar un nodo, se verifica si size supera maxSize. Si lo supera, se elimina automáticamente el nodo más antiguo. Esto cumple el requisito de alternar entre 50 y 100 trazos, manteniendo siempre la cola dentro del límite definido.

Evidencia 5: Recorrido de la cola sin destruirla (draw)
Node* current = strokes.front;
while (current != nullptr) {
    float ageFade = ofMap(index, 0, strokes.size - 1, 60, 255);
    ofSetColor(current->color, ageFade);
    ofDrawCircle(current->x, current->y, current->radius);
    current = current->next;
    index++;
}
Se usa un puntero auxiliar current para recorrer la cola desde front hasta nullptr, sin modificar ni eliminar ningún nodo. La cola queda intacta después del recorrido. Además, ageFade hace que los trazos más antiguos (índice bajo) tengan menos opacidad, cumpliendo el efecto de pintura dinámica del enunciado.

Evidencia 6: Limpieza total de la memoria (clear)
void BrushQueue::clear() {
    while (!isEmpty()) {
        dequeue();
    }
}
Llama a dequeue() repetidamente hasta que la cola esté vacía. Como dequeue() hace delete en cada nodo, todos los nodos son liberados correctamente. Esto cumple el requisito de gestionar la memoria sin fugas, y es también lo que ejecuta el destructor ~BrushQueue() automáticamente al finalizar el programa.

## Bitácora de reflexión
