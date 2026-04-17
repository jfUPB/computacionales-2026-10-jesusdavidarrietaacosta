# Unidad 6
## Bitácora de aplicación 

### OffApp.cpp

```
#include "ofApp.h"
#include <algorithm>

// ================= SUBJECT =================
void Subject::addObserver(Observer * observer) {
	if (!observer) return;
	if (std::find(observers.begin(), observers.end(), observer) == observers.end()) {
		observers.push_back(observer);
	}
}

void Subject::removeObserver(Observer * observer) {
	if (!observer) return;
	observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end());
}

void Subject::notify(const std::string & event) {
	for (Observer * observer : observers) {
		observer->onNotify(event);
	}
}

// ================= PARTICLE =================
Particle::Particle()
	: state(nullptr) {
	position = ofVec2f(ofRandomWidth(), ofRandomHeight());
	velocity = ofVec2f(ofRandom(-0.5f, 0.5f), ofRandom(-0.5f, 0.5f));
	size = ofRandom(2.0f, 5.0f);
	color = ofColor(255);

	state = new NormalState();
	state->onEnter(this);
}

Particle::~Particle() {
	if (state) {
		state->onExit(this);
		delete state;
	}
}

void Particle::setState(State * newState) {
	if (state) {
		state->onExit(this);
		delete state;
	}
	state = newState;
	if (state) {
		state->onEnter(this);
	}
}

void Particle::update() {
	if (state) {
		state->update(this);
	}
	keepInsideWindow();
}

void Particle::draw() {
	ofPushStyle();
	ofSetColor(color);
	ofDrawCircle(position, size);
	ofPopStyle();
}

void Particle::onNotify(const std::string & event) {
	if (event == "attract") {
		setState(new AttractState());

	} else if (event == "repel") {
		setState(new RepelState());

	} else if (event == "stop") {
		setState(new StopState());

	} else if (event == "normal") {
		setState(new NormalState());

	}
	//  NUEVO EVENTO
	else if (event == "fall") {
		setState(new FallState());
	}
}

void Particle::keepInsideWindow() {
	float W = ofGetWidth();
	float H = ofGetHeight();

	if (position.x < 0) {
		position.x = 0;
		velocity.x *= -1;
	}
	if (position.x > W) {
		position.x = W;
		velocity.x *= -1;
	}
	if (position.y < 0) {
		position.y = 0;
		velocity.y *= -1;
	}
	if (position.y > H) {
		position.y = H;
		velocity.y *= -1;
	}
}

// ================= STATES =================
void NormalState::onEnter(Particle * particle) {
	particle->velocity.set(ofRandom(-0.5f, 0.5f), ofRandom(-0.5f, 0.5f));
}

void NormalState::update(Particle * particle) {
	particle->position += particle->velocity;
}

static void steer(Particle * particle, const ofVec2f & target, float accel, float vmax, float scale) {
	ofVec2f dir = target - particle->position;
	float len = dir.length();

	if (len > 1e-6f) {
		dir /= len;
		particle->velocity += dir * accel;
	}

	particle->velocity.limit(vmax);
	particle->position += particle->velocity * scale;
}

void AttractState::update(Particle * particle) {
	ofVec2f mouse(ofGetMouseX(), ofGetMouseY());
	steer(particle, mouse, 0.05f, 3.0f, 0.2f);
}

void RepelState::update(Particle * particle) {
	ofVec2f mouse(ofGetMouseX(), ofGetMouseY());
	ofVec2f dir = particle->position - mouse;

	float len = dir.length();
	if (len > 1e-6f) {
		dir /= len;
		particle->velocity += dir * 0.05f;
	}

	particle->velocity.limit(3.0f);
	particle->position += particle->velocity * 0.2f;
}

void StopState::update(Particle * particle) {
	particle->velocity *= 0.8f;
	if (particle->velocity.lengthSquared() < 1e-4f) {
		particle->velocity.set(0, 0);
	}
	particle->position += particle->velocity;
}

//  NUEVO ESTADO (CAÍDA)
void FallState::update(Particle * particle) {
	particle->velocity.y += 0.2f;

	if (particle->velocity.y > 5.0f) {
		particle->velocity.y = 5.0f;
	}

	particle->position += particle->velocity;

	particle->color = ofColor(100, 150, 255);

	if (particle->position.y > ofGetHeight()) {
		particle->position.y = 0;
		particle->position.x = ofRandomWidth();
		particle->velocity.y = 0;
	}
}

// ================= FACTORY =================
Particle * ParticleFactory::createParticle(const std::string & type) {
	Particle * particle = new Particle();

	if (type == "star") {
		particle->size = ofRandom(2.0f, 4.0f);
		particle->color = ofColor(255, 0, 0);

	} else if (type == "shooting_star") {
		particle->size = ofRandom(3.0f, 6.0f);
		particle->color = ofColor(0, 255, 0);
		particle->velocity *= 3.0f;

	} else if (type == "planet") {
		particle->size = ofRandom(5.0f, 8.0f);
		particle->color = ofColor(0, 0, 255);

	}
	//  NUEVO TIPO (LLUVIA)
	else if (type == "comet") {
		particle->size = ofRandom(2.0f, 4.0f);
		particle->color = ofColor(100, 150, 255);
		particle->velocity = ofVec2f(0, ofRandom(1.0f, 3.0f));
	}


	return particle;
}

// ================= APP =================
ofApp::~ofApp() {
	for (Particle * p : particles) {
		removeObserver(p);
		delete p;
	}
	particles.clear();
}

void ofApp::setup() {
	ofBackground(0);

	for (int i = 0; i < 100; i++) {
		Particle * p = ParticleFactory::createParticle("star");
		particles.push_back(p);
		addObserver(p);
	}

	for (int i = 0; i < 5; i++) {
		Particle * p = ParticleFactory::createParticle("shooting_star");
		particles.push_back(p);
		addObserver(p);
	}

	for (int i = 0; i < 10; i++) {
		Particle * p = ParticleFactory::createParticle("planet");
		particles.push_back(p);
		addObserver(p);
	}

	//  NUEVAS PARTÍCULAS (LLUVIA)
	for (int i = 0; i < 20; i++) {
		Particle * p = ParticleFactory::createParticle("comet");
		particles.push_back(p);
		addObserver(p);
	}
}

void ofApp::update() {
	for (Particle * p : particles) {
		p->update();
	}
}

void ofApp::draw() {
	for (Particle * p : particles) {
		p->draw();
	}
}

void ofApp::keyPressed(int key) {
	switch (key) {
	case 's':
		notify("stop");
		break;
	case 'a':
		notify("attract");
		break;
	case 'r':
		notify("repel");
		break;
	case 'n':
		notify("normal");
		break;

	//  NUEVA TECLA
	case 'f':
		notify("fall");
		break;

	default:
		break;
	}
}
```

### ofApp.h
```
#pragma once

#include "ofMain.h"
#include <string>
#include <vector>

// ================= OBSERVER =================
class Observer {
public:
	virtual ~Observer() = default;
	virtual void onNotify(const std::string & event) = 0;
};

class Subject {
public:
	void addObserver(Observer * observer);
	void removeObserver(Observer * observer);

protected:
	void notify(const std::string & event);

private:
	std::vector<Observer *> observers;
};

// ================= STATE =================
class Particle;

class State {
public:
	virtual ~State() = default;
	virtual void update(Particle * particle) = 0;
	virtual void onEnter(Particle * particle) { }
	virtual void onExit(Particle * particle) { }
};

// ================= PARTICLE =================
class Particle : public Observer {
public:
	Particle();
	~Particle() override;

	Particle(const Particle &) = delete;
	Particle & operator=(const Particle &) = delete;

	void update();
	void draw();
	void onNotify(const std::string & event) override;

	void setState(State * newState);

	ofVec2f position;
	ofVec2f velocity;
	float size;
	ofColor color;

private:
	void keepInsideWindow();
	State * state;
};

// ================= STATES =================
class NormalState : public State {
public:
	void update(Particle * particle) override;
	void onEnter(Particle * particle) override;
};

class AttractState : public State {
public:
	void update(Particle * particle) override;
};

class RepelState : public State {
public:
	void update(Particle * particle) override;
};

class StopState : public State {
public:
	void update(Particle * particle) override;
};

//  NUEVO ESTADO
class FallState : public State {
public:
	void update(Particle * particle) override;
};

// ================= FACTORY =================
class ParticleFactory {
public:
	static Particle * createParticle(const std::string & type);
};

// ================= APP =================
class ofApp : public ofBaseApp, public Subject {
public:
	~ofApp() override;
	void setup() override;
	void update() override;
	void draw() override;
	void keyPressed(int key) override;

private:
	std::vector<Particle *> particles;
};

```

Evidenvia 1: 
En la siguiene captura podemos apreciar como se crea una nueva particula (comet) de tipo Particle* y con tamaño valor 3.614..., color	{r=100 'd' g=150 '–' b=255 'ÿ' ...}	size	3.75480986, velocity	x=-0.411335230 y=0.325940788 

<img width="1675" height="948" alt="Captura de pantalla 2026-04-17 103737" src="https://github.com/user-attachments/assets/c9afd9b5-1aab-45ae-ada9-cffa4b35faf1" />

Evidencia 2: 
Al momento de expandir el state podemos apreciar como este se encuentra en una direccion diferente y el vfptr es un puntero a vtable

<img width="1673" height="876" alt="Captura de pantalla 2026-04-17 105615" src="https://github.com/user-attachments/assets/b8f3fcdb-96e8-4ff8-8931-fd3ecee02ef4" />
<img width="1675" height="972" alt="Captura de pantalla 2026-04-17 105452" src="https://github.com/user-attachments/assets/5d0ee4ea-ad0e-4448-9ea3-b4adb4ffc647" />








