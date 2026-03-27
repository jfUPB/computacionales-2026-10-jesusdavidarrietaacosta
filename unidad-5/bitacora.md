# Codigo

offApp.cpp


```
#include "ofApp.h"

 void ofApp::setup() 
 
 {
	ofSetFrameRate(60);
	ofBackground(0);
}

void ofApp::update() 

{
	float dt = ofGetLastFrameTime();

	for (int i = 0; i < (int)particles.size(); i++) {
		particles[i]->update(dt);
	}

	for (int i = (int)particles.size() - 1; i >= 0; i--) {
		if (particles[i]->shouldExplode()) {
			
			int explosionType = (int)ofRandom(4);
			int numParticles = (int)ofRandom(20, 30);

			glm::vec2 pos = particles[i]->getPosition();
			ofColor col = particles[i]->getColor();

			for (int j = 0; j < numParticles; j++) {
				switch (explosionType) {
				case 0:
					particles.push_back(new CircularExplosion(pos, col));
					break;
				case 1:
					particles.push_back(new RandomExplosion(pos, col));
					break;
				case 2:
					particles.push_back(new StarExplosion(pos, col));
					break;
				case 3:
					particles.push_back(new ExplosionCreciente(pos, col));
					break;
				}
			}

			delete particles[i];
			particles.erase(particles.begin() + i);

		} else if (particles[i]->isDead()) {
			delete particles[i];
			particles.erase(particles.begin() + i);
		}
	}
}


void ofApp::draw() 

{
	for (int i = 0; i < (int)particles.size(); i++) {
		particles[i]->draw();
	}
}


void ofApp::createRisingParticle() 

{
	float minX = ofGetWidth() * 0.35f;
	float maxX = ofGetWidth() * 0.65f;
	float spawnX = ofRandom(minX, maxX);

	glm::vec2 pos(spawnX, ofGetHeight());
	glm::vec2 target(ofGetWidth() / 2.0f + ofRandom(-300, 300),
		ofGetHeight() * 0.10f + ofRandom(-30, 30));
	glm::vec2 vel = glm::normalize(target - pos) * ofRandom(250, 350);

	ofColor col;
	col.setHsb(ofRandom(255), 220, 255);
	float lifetime = ofRandom(1.5f, 3.5f);

	
	int rocketType = (int)ofRandom(3);
	switch (rocketType) {
	case 0:
		particles.push_back(new RisingParticle(pos, vel, col, lifetime));
		break;
	case 1:
		particles.push_back(new HexagonalRisingParticle(pos, vel, col, lifetime));
		break;
	case 2:
		particles.push_back(new TriangularRisingParticle(pos, vel, col, lifetime));
		break;
	}
}


void ofApp::mousePressed(int x, int y, int button) 

{
	createRisingParticle();
}


void ofApp::keyPressed(int key) 

{
	if (key == ' ') {
		for (int i = 0; i < 1000; i++) 
    
    {
			createRisingParticle();
		}
	}
	if (key == 's') 
  
  {
		ofSaveScreen("screenshot_" + ofToString(ofGetFrameNum()) + ".png");
	}
}


ofApp::~ofApp() 

{
	for (int i = 0; i < (int)particles.size(); i++) {
		delete particles[i];
	}
	particles.clear();
}

```



offApp.h

```

#pragma once
#include "ofMain.h"
#include <vector>


class Particle {
public:
	virtual ~Particle() { }
	virtual void update(float dt) = 0;
	virtual void draw() = 0;
	virtual bool isDead() const = 0;
	virtual bool shouldExplode() const { return false; }
	virtual glm::vec2 getPosition() const { return glm::vec2(0, 0); }
	virtual ofColor getColor() const { return ofColor(255); }
};


class RisingParticle : public Particle {
protected:
	glm::vec2 position;
	glm::vec2 velocity;
	ofColor color;
	float lifetime;
	float age;
	bool exploded;

public:
	RisingParticle(const glm::vec2 & pos, const glm::vec2 & vel,
		const ofColor & col, float life)
		: position(pos)
		, velocity(vel)
		, color(col)
		, lifetime(life)
		, age(0)
		, exploded(false) { }

	void update(float dt) override {
		position += velocity * dt;
		age += dt;
		velocity.y += 9.8f * dt * 8;
		float explosionThreshold = ofGetHeight() * 0.15f + ofRandom(-30, 30);
		if (position.y <= explosionThreshold || age >= lifetime) {
			exploded = true;
		}
	}

	void draw() override {
		ofSetColor(color);
		ofDrawCircle(position, 10);
	}

	bool isDead() const override { return exploded; }
	bool shouldExplode() const override { return exploded; }
	glm::vec2 getPosition() const override { return position; }
	ofColor getColor() const override { return color; }
};


class HexagonalRisingParticle : public RisingParticle {
public:
	HexagonalRisingParticle(const glm::vec2 & pos, const glm::vec2 & vel,
		const ofColor & col, float life)
		: RisingParticle(pos, vel, col, life) { }

	void draw() override {
		ofSetColor(color);
		ofPushMatrix();
		ofTranslate(position);

		
		ofPath hex;
		hex.setFilled(false);
		const int sides = 6;
		const float radius = 10.0f;
		for (int i = 0; i < sides; i++) {
			float angle = TWO_PI / sides * i - PI / 6.0f;
			float x = cos(angle) * radius;
			float y = sin(angle) * radius;
			if (i == 0)
				hex.moveTo(x, y);
			else
				hex.lineTo(x, y);
		}
		hex.close();
		hex.setStrokeColor(color);
		hex.setStrokeWidth(2);
		hex.draw();

		ofPopMatrix();
	}
};


class TriangularRisingParticle : public RisingParticle {
public:
	TriangularRisingParticle(const glm::vec2 & pos, const glm::vec2 & vel,
		const ofColor & col, float life)
		: RisingParticle(pos, vel, col, life) { }

	void draw() override {
		ofSetColor(color);
		ofPushMatrix();
		ofTranslate(position);

		
		ofPath tri;
		tri.setFilled(false);
		const int sides = 3;
		const float radius = 10.0f;
		for (int i = 0; i < sides; i++) {
			
			float angle = TWO_PI / sides * i - PI / 2.0f;
			float x = cos(angle) * radius;
			float y = sin(angle) * radius;
			if (i == 0)
				tri.moveTo(x, y);
			else
				tri.lineTo(x, y);
		}
		tri.close();
		tri.setStrokeColor(color);
		tri.setStrokeWidth(2);
		tri.draw();

		ofPopMatrix();
	}
};


class ExplosionParticle : public Particle {
protected:
	glm::vec2 position;
	glm::vec2 velocity;
	ofColor color;
	float age;
	float lifetime;
	float size;

public:
	ExplosionParticle(const glm::vec2 & pos, const glm::vec2 & vel,
		const ofColor & col, float life, float sz)
		: position(pos)
		, velocity(vel)
		, color(col)
		, age(0)
		, lifetime(life)
		, size(sz) { }

	void update(float dt) override {
		position += velocity * dt;
		age += dt;
		float alpha = ofMap(age, 0, lifetime, 255, 0, true);
		color.a = alpha;
	}

	bool isDead() const override { return age >= lifetime; }
};


class CircularExplosion : public ExplosionParticle {
public:
	CircularExplosion(const glm::vec2 & pos, const ofColor & col)
		: ExplosionParticle(pos, glm::vec2(0, 0), col, 1.2f, ofRandom(16, 32)) {
		float angle = ofRandom(0, TWO_PI);
		float speed = ofRandom(80, 200);
		velocity = glm::vec2(cos(angle), sin(angle)) * speed;
	}

	void draw() override {
		ofSetColor(color);
		ofDrawCircle(position, size);
	}
};


class RandomExplosion : public ExplosionParticle {
public:
	RandomExplosion(const glm::vec2 & pos, const ofColor & col)
		: ExplosionParticle(pos, glm::vec2(0, 0), col, 1.5f, ofRandom(16, 32)) {
		velocity = glm::vec2(ofRandom(-200, 200), ofRandom(-200, 200));
	}

	void draw() override {
		ofSetColor(color);
		ofDrawRectangle(position.x, position.y, size, size);
	}
};


class StarExplosion : public ExplosionParticle {
public:
	StarExplosion(const glm::vec2 & pos, const ofColor & col)
		: ExplosionParticle(pos, glm::vec2(0, 0), col, 1.3f, ofRandom(20, 40)) {
		float angle = ofRandom(0, TWO_PI);
		float speed = ofRandom(90, 180);
		velocity = glm::vec2(cos(angle), sin(angle)) * speed;
	}

	void draw() override {
		ofSetColor(color);
		int rays = 5;
		float outerRadius = size;
		float innerRadius = size * 0.5f;
		ofPushMatrix();
		ofTranslate(position);
		for (int i = 0; i < rays; i++) {
			float theta = ofMap(i, 0, rays, 0, TWO_PI);
			float xOuter = cos(theta) * outerRadius;
			float yOuter = sin(theta) * outerRadius;
			float xInner = cos(theta + PI / rays) * innerRadius;
			float yInner = sin(theta + PI / rays) * innerRadius;
			ofDrawLine(0, 0, xOuter, yOuter);
			ofDrawLine(xOuter, yOuter, xInner, yInner);
		}
		ofPopMatrix();
	}
};

class ExplosionCreciente : public ExplosionParticle {
private:
	float maxSize; 
	float growSpeed; 
public:
	ExplosionCreciente(const glm::vec2 & pos, const ofColor & col)
		
		: ExplosionParticle(pos, glm::vec2(0, 0), col, 99.0f, 0.0f) {
		maxSize = ofRandom(60, 140);
		
		growSpeed = maxSize;
	}

	void update(float dt) override {
		age += dt;
		size += growSpeed * dt; 

		
		float alpha = ofMap(size, 0, maxSize, 255, 0, true);
		color.a = (unsigned char)alpha;
	}

	bool isDead() const override { return size >= maxSize; }

	void draw() override {
		ofSetColor(color);
		ofNoFill();
		ofSetLineWidth(3);
		ofDrawCircle(position, size);
		ofFill();
		ofSetLineWidth(1); 
	}
};


class ofApp : public ofBaseApp {
public:
	void setup();
	void update();
	void draw();
	void mousePressed(int x, int y, int button);
	void keyPressed(int key);

	std::vector<Particle *> particles;
	~ofApp();

private:
	void createRisingParticle();
};
```






# Evidencia 

## Evidencia 1.
<img width="1654" height="881" alt="Captura de pantalla 2026-03-27 083931" src="https://github.com/user-attachments/assets/761829d2-34fd-44f2-bc62-c211057c942c" />

Aqui como podemos ver sucede la herencia, siendo que para la HexagonalRisingParticle va a tener elementos herdados de la RisingParticle, que a su vez esta tiene herencia de Particle. Unas lineas mas adelante podemos apresiar el "RisingParticle(pos, vel, col, life)" que es un inicializador de sub objeto. Le pasa los 4 parámetros al constructor del padre para que él inicialice sus propios campos. El inicializador: RisingParticle(...) demuestra que los objetos se construyen de arriba hacia abajo en la jerarquía. HexagonalRisingParticle no puede inicializar position o color directamente porque no los declaró tiene que pedirle al padre que lo haga. Esto evidencia que el subobjeto padre existe físicamente dentro del hijo.

## Evidencia 2.
<img width="1675" height="932" alt="Captura de pantalla 2026-03-27 100332" src="https://github.com/user-attachments/assets/60ee8d1d-4949-4ff4-91d1-d6efcf85a4c2" />
<img width="1646" height="904" alt="Captura de pantalla 2026-03-27 101929" src="https://github.com/user-attachments/assets/57fc390e-0447-4ec8-8353-ec4d481055cd" />

En ambas imagenes se muestra las dos partes del codigo donde podemos observar que ambas las partes iguales del codigo son aquellas en las cuales se aprecia que heredaron de explosionparticule. Cada clase concreta tiene su propia vtable generada por el compilador. La vtable es un array de punteros a función que se construye en tiempo de compilación y se referencia desde el objeto mediante el vptr (primer campo oculto del objeto).

## Evidencia 3.
El polimorfismo en tiempo de ejecución se evidencia en la llamada particles[i]->update(dt) dentro de ofApp::update(), donde un puntero de tipo Particle* invoca diferentes implementaciones del método virtual según el tipo real del objeto.


## Evidencia 4.
<img width="1644" height="878" alt="Captura de pantalla 2026-03-27 105042" src="https://github.com/user-attachments/assets/d0ab824a-03a4-4b8d-87ce-a73b1c785f3a" />

En esta primera captura podemos ver como las funciones protected de la class explosionparticule

<img width="1639" height="876" alt="Captura de pantalla 2026-03-27 105322" src="https://github.com/user-attachments/assets/9f8f4794-40b2-4a61-bd35-5a94a2373637" />

Y en esta la subclase "explosioncreciente" donde en la parte del 
```
void update(float dt) override {
	age += dt;
	size += growSpeed * dt; 
	float alpha = ofMap(size, 0, maxSize, 255, 0, true);
	color.a = (unsigned char)alpha;
```
Nos muentra que age, size, color vienen de la clase base por lo tanto hay encapsulamiento.

Ademas en el depurador, al inspeccionar el objeto (this), se observan todos los atributos —incluyendo los privados—, lo que demuestra que el encapsulamiento es una restricción a nivel de compilación y diseño, no una limitación de la memoria en tiempo de ejecución.

## Evidencia 5
<img width="1645" height="645" alt="Captura de pantalla 2026-03-27 111153" src="https://github.com/user-attachments/assets/b90fab25-b117-469c-8293-a6c470e5113c" />
El ciclo de vida de ExplosionCreciente comienza en ofApp::update(), cuando se crea dinámicamente con new ExplosionCreciente(pos, col) y se inserta en el vector particles mediante push_back, lo que evidencia su almacenamiento como un Particle*.

<img width="1636" height="872" alt="Captura de pantalla 2026-03-27 112110" src="https://github.com/user-attachments/assets/5a096aef-501e-4134-ad8a-090ae211cd25" />
Durante su vida, el objeto es actualizado en la llamada particles[i]->update(dt), donde, mediante polimorfismo, se ejecuta ExplosionCreciente::update.

<img width="1645" height="844" alt="Captura de pantalla 2026-03-27 112804" src="https://github.com/user-attachments/assets/e5ab9aec-a639-49a4-9c75-82147d25e4e0" />
El objeto alcanza su condición de muerte cuando particles[i]->isDead() retorna verdadero, lo cual ocurre cuando size >= maxSize, reflejando una lógica distinta a la clase base.

<img width="1645" height="849" alt="Captura de pantalla 2026-03-27 112913" src="https://github.com/user-attachments/assets/7305c908-a8fa-4056-a89a-aaa9ff94c185" />
Finalmente, en el mismo método update, el objeto es eliminado mediante delete particles[i], lo que invoca correctamente la cadena de destructores gracias al uso de un destructor virtual, y posteriormente es removido del vector con erase, liberando tanto la memoria como la referencia en la estructura de datos

## Evidencia 6

No existe fuga de memoria en el programa, ya que cada objeto creado dinámicamente con new es eliminado correctamente mediante delete cuando cumple su condición de muerte (isDead()).
En el código, esto ocurre en el método ofApp::update(), donde primero se libera la memoria del objeto con delete particles[i]
<img width="1647" height="844" alt="image" src="https://github.com/user-attachments/assets/cc9386d9-f519-4356-a902-1aede52fcf7b" />
lo que invoca el destructor virtual correspondiente (~ExplosionCreciente → ~ExplosionParticle → ~Particle), asegurando una destrucción completa del objeto.

Posteriormente, el puntero es eliminado del vector mediante particles.erase(particles.begin() + i), lo que garantiza que no queden referencias al objeto eliminado.
<img width="1646" height="842" alt="image" src="https://github.com/user-attachments/assets/6249d785-0a4e-4d6d-904c-0e987ba1ad66" />

Adicionalmente, el destructor de ofApp recorre el vector y elimina cualquier objeto restante, asegurando que no queden fugas de memoria al finalizar la ejecución del programa.

## Evidencia 7
Se tomo el de diseño de estres ya implementado en el cual se crean 1000 partículas simultáneamente al presionar la tecla espacio utilizando el método createRisingParticle(), pero par ver que tanto aguanta el programa se sometera a varias cantiades de particulas cada vez mayores. Se tomaran dos capturas por cantidad una para la particulas, y otra para la explosion

1000:
particulas:
<img width="2704" height="1050" alt="Captura de pantalla (18)" src="https://github.com/user-attachments/assets/fe350e7b-0613-42d5-9f19-5430caee8508" />

Explosion:
<img width="2704" height="1050" alt="Captura de pantalla (20)" src="https://github.com/user-attachments/assets/3fc59b7b-4b2a-4f0b-9363-6c241db36705" />

5000:
Particulas:
<img width="2704" height="1050" alt="Captura de pantalla (21)" src="https://github.com/user-attachments/assets/f3abed3e-af34-47c0-8d5d-3ae5db4f6f6d" />

Explosion:
<img width="2704" height="1050" alt="Captura de pantalla (22)" src="https://github.com/user-attachments/assets/60b2aeca-14b6-49c9-8b47-436218b13ab3" />

10000:
Particulas:
<img width="2704" height="1050" alt="Captura de pantalla (23)" src="https://github.com/user-attachments/assets/de94b747-d8db-432a-8f4a-e1b898801c93" />

Explosion:
<img width="2704" height="1050" alt="Captura de pantalla (25)" src="https://github.com/user-attachments/assets/bd23428d-3ef7-44a6-a020-e53bbaa90139" />

Este escenario es relevante porque permite evaluar el comportamiento del sistema ante una carga alta de objetos dinámicos, verificando tanto la creación como la destrucción de los mismos.
