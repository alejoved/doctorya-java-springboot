Fase 1: Fundamentos y Diagnóstico (Semanas 1-2)
1. Mapeo del Panorama Actual
Inventario de sistemas y tecnologías por línea de producto
Identificar cuáles tienen mayor criticidad y volumen de pruebas manuales
Evaluar madurez actual: ¿hay pruebas manuales documentadas? ¿Hay casos de regresión?
Recursos disponibles: herramientas, infraestructura, presupuesto

Fase 2: Definir Estándares Centralizados (Semanas 2-3)
Aunque hay múltiples tecnologías, necesitas principios comunes:

1. Estructura de Proyectos de Pruebas

producto-automation/
├── tests/
│   ├── unit/           # Pruebas unitarias
│   ├── integration/    # APIs, bases de datos
│   ├── e2e/           # Flujos completos
│   └── smoke/         # Pruebas críticas rápidas
├── config/
│   ├── environments.yaml
│   └── test-data/
├── docs/
│   ├── SETUP.md
│   └── STANDARDS.md
└── CI/CD/
    └── .github/workflows/ o pipeline definitions


2. Estándares de Naming y Nomenclatura
Define convenciones claras (aunque cambies lenguajes):
test_[feature]_[scenario]_[expected_result]
test_login_valid_credentials_returns_token
test_payment_invalid_amount_shows_error

3. Estructura de Datos de Prueba
Centralizar test data (fixtures)
Versionado de datos de prueba
Ambientes: DEV, QA, STAGING, PRODUCTION

Fase 3: Seleccionar Herramientas por Capa
Layer: Pruebas Unitarias
Tecnología	Herramienta	Enfoque
Java Backend	JUnit 5 + Mockito	Standard
Python Backend	pytest	Standard
Node.js Backend	Jest	Standard
C# Backend	xUnit	Standard
AS400 (RPG/COBOL)	JUnit	Standard

Layer: Integración / APIs
Estándar: REST Assured, Postman, Insomnia
- REST Assured (Java)
- Requests (Python) o Cypress/Playwright (TypeScript)
- RestSharp (C#)
Datos: JSON fixtures, faker libraries

Layer: E2E / UI
Frontend Web:
  - Selenium (Legacy, amplio soporte)
  - Cypress (Recomendado: mejor DX, debugging)
  - Playwright (Moderno: multi-browser, Python/TypeScript support)

Frontend Mobile:
  - Appium (multiplataforma)
  - Espresso (Android native)
  - XCUITest (iOS native)

AS400 Terminal:
  - Speciality: Approaches como screen scraping con herramientas como:
  - IBM RFT (Rational Functional Tester)
  - AutoIT (si es terminal emulada)
  - Captura de pantallas + OCR para validaciones

Fase 4: Planificación Piloto
No automatices todo. Elige un producto piloto que:

✅ Sea crítico para el negocio
✅ Tenga flujos claros y documentados
✅ No sea demasiado complejo (evita AS400 como primer piloto)
✅ Tenga soporte del equipo
Ejemplo de piloto: Backend API + Frontend Web (sin AS400 aún)

Fase 5: Implementación del Piloto

Paso 1: Setup de Infraestructura
1. Repository centralizado (GitHub, GitLab, Bitbucket)
2. CI/CD pipeline básico (GitHub Actions, Jenkins, GitLab CI)
3. Test environment estable
4. Reporte centralizado (Allure, TestRail)

Paso 2: Crear Suite Base
Empieza con:

5-10 pruebas unitarias del backend
5-10 pruebas de API
5-10 pruebas E2E del flujo crítico
Objetivo: Demostrar valor rápidamente

Herramientas Centralizadas (que usarán todos)
Repositorio: GitHub/GitLab (monorepo o multi-repo organizado)
CI/CD: GitHub Actions o Jenkins
Reporte: Allure Report o TestRail
Monitoreo: Dashboard centralizado (Grafana o custom)
Documentación: Confluence o Wiki en el repo
Comunicación: Slack + ceremonias semanales

Escenario con AS400
Crear un wrapper Java que llame RPG/COBOL
Expones el código legado como servicios que Java puede llamar:
// Java wrapper
public class RPGWrapper {
    
    @Test
    public void testRPGFunction() {
        // Llama el programa RPG desde Java
        ProgramCall pc = new ProgramCall(as400Connection);
        pc.setProgram("/QSYS.LIB/TULIB.LIB/MYPROG.PGM");
        
        ProgramParameter[] params = new ProgramParameter[2];
        params[0] = new ProgramParameter(100);      // input
        params[1] = new ProgramParameter(10);       // output
        
        pc.run(params);
        
        byte[] output = params[1].getOutputData();
        int resultado = BinaryConverter.byteArrayToInt(output);
        
        assertEquals(110, resultado);
    }
}

Setup Técnico Específico
Si quieres integrar JUnit con AS400, necesitas:

JT400 Library (Toolbox para Java - IBM)
<dependency>
    <groupId>com.ibm.as400</groupId>
    <artifactId>jt400</artifactId>
    <version>11.0</version>
</dependency>

Connection Pool a AS400
public class AS400ConnectionPool {
    private static AS400ConnectionPool pool;
    
    public static AS400 getConnection() {
        AS400 as400 = new AS400("hostname", "user", "password");
        return as400;
    }
}

Test Base Class
public abstract class AS400TestBase {
    protected AS400 as400;
    
    @Before
    public void setup() {
        as400 = AS400ConnectionPool.getConnection();
    }
    
    @After
    public void teardown() {
        as400.disconnectAllServices();
    }
}

2. PRUEBAS UNITARIAS - BACKEND
Escenario A: Spring Boot (Java)
// pom.xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>3.0.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.2.0</version>
</dependency>

// UserService.java - Servicio a probar
public class UserService {
    private UserRepository userRepository;
    private EmailService emailService;
    
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public User createUser(String email, String name) {
        if (email == null || email.isEmpty()) {
            throw new IllegalArgumentException("Email cannot be empty");
        }
        
        User user = new User(email, name);
        userRepository.save(user);
        emailService.sendWelcomeEmail(email);
        
        return user;
    }
    
    public User getUserById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found"));
    }
}

// UserServiceTest.java - Pruebas unitarias
import static org.mockito.Mockito.*;
import static org.junit.jupiter.api.Assertions.*;

public class UserServiceTest {
    
    private UserService userService;
    private UserRepository userRepository;
    private EmailService emailService;
    
    @BeforeEach
    void setUp() {
        // Crear mocks
        userRepository = mock(UserRepository.class);
        emailService = mock(EmailService.class);
        
        // Inyectar mocks en el servicio
        userService = new UserService(userRepository, emailService);
    }
    
    @Test
    void testCreateUserSuccess() {
        // Arrange
        String email = "juan@example.com";
        String name = "Juan Pérez";
        User expectedUser = new User(email, name);
        
        when(userRepository.save(any(User.class))).thenReturn(expectedUser);
        
        // Act
        User result = userService.createUser(email, name);
        
        // Assert
        assertNotNull(result);
        assertEquals(email, result.getEmail());
        assertEquals(name, result.getName());
        
        // Verificar que se llamaron los métodos correctos
        verify(userRepository, times(1)).save(any(User.class));
        verify(emailService, times(1)).sendWelcomeEmail(email);
    }
    
    @Test
    void testCreateUserWithEmptyEmailThrowsException() {
        // Act & Assert
        assertThrows(IllegalArgumentException.class, () -> {
            userService.createUser("", "Juan");
        });
        
        // Verificar que NO se guardó ni se envió email
        verify(userRepository, never()).save(any());
        verify(emailService, never()).sendWelcomeEmail(anyString());
    }
    
    @Test
    void testGetUserByIdNotFound() {
        // Arrange
        when(userRepository.findById(999L)).thenReturn(Optional.empty());
        
        // Act & Assert
        assertThrows(UserNotFoundException.class, () -> {
            userService.getUserById(999L);
        });
    }
}

Escenario B: Node.js / Express (Jest)
// package.json
{
  "devDependencies": {
    "jest": "^29.5.0",
    "supertest": "^6.3.3"
  }
}

// userService.js
class UserService {
  constructor(userRepository, emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }

  async createUser(email, name) {
    if (!email || email.trim() === '') {
      throw new Error('Email cannot be empty');
    }

    const user = { email, name };
    await this.userRepository.save(user);
    await this.emailService.sendWelcomeEmail(email);

    return user;
  }

  async getUserById(id) {
    const user = await this.userRepository.findById(id);
    if (!user) {
      throw new Error('User not found');
    }
    return user;
  }
}

module.exports = UserService;

// userService.test.js - Pruebas unitarias
const UserService = require('./userService');

describe('UserService', () => {
  let userService;
  let userRepository;
  let emailService;

  beforeEach(() => {
    // Crear mocks
    userRepository = {
      save: jest.fn(),
      findById: jest.fn(),
    };

    emailService = {
      sendWelcomeEmail: jest.fn(),
    };

    userService = new UserService(userRepository, emailService);
  });

  test('should create user successfully', async () => {
    // Arrange
    const email = 'juan@example.com';
    const name = 'Juan Pérez';
    userRepository.save.mockResolvedValue({ email, name });

    // Act
    const result = await userService.createUser(email, name);

    // Assert
    expect(result.email).toBe(email);
    expect(result.name).toBe(name);
    expect(userRepository.save).toHaveBeenCalledWith({ email, name });
    expect(emailService.sendWelcomeEmail).toHaveBeenCalledWith(email);
  });

  test('should throw error when email is empty', async () => {
    // Act & Assert
    await expect(userService.createUser('', 'Juan')).rejects.toThrow(
      'Email cannot be empty'
    );

    expect(userRepository.save).not.toHaveBeenCalled();
    expect(emailService.sendWelcomeEmail).not.toHaveBeenCalled();
  });

  test('should throw error when user not found', async () => {
    // Arrange
    userRepository.findById.mockResolvedValue(null);

    // Act & Assert
    await expect(userService.getUserById(999)).rejects.toThrow('User not found');
  });
});

Escenario C: Python / FastAPI (pytest)
# requirements-dev.txt
pytest==7.3.1
pytest-asyncio==0.21.0
pytest-mock==3.10.0

# user_service.py
class UserService:
    def __init__(self, user_repository, email_service):
        self.user_repository = user_repository
        self.email_service = email_service

    def create_user(self, email: str, name: str) -> dict:
        if not email or email.strip() == '':
            raise ValueError('Email cannot be empty')

        user = {'email': email, 'name': name}
        self.user_repository.save(user)
        self.email_service.send_welcome_email(email)

        return user

    def get_user_by_id(self, user_id: int) -> dict:
        user = self.user_repository.find_by_id(user_id)
        if not user:
            raise Exception('User not found')
        return user

# test_user_service.py - Pruebas unitarias
import pytest
from unittest.mock import Mock, patch
from user_service import UserService

class TestUserService:
    
    @pytest.fixture
    def setup(self):
        """Setup para cada test"""
        self.user_repository = Mock()
        self.email_service = Mock()
        self.user_service = UserService(
            self.user_repository, 
            self.email_service
        )

    def test_create_user_success(self, setup):
        # Arrange
        email = 'juan@example.com'
        name = 'Juan Pérez'

        # Act
        result = self.user_service.create_user(email, name)

        # Assert
        assert result['email'] == email
        assert result['name'] == name
        self.user_repository.save.assert_called_once()
        self.email_service.send_welcome_email.assert_called_once_with(email)

    def test_create_user_empty_email(self, setup):
        # Act & Assert
        with pytest.raises(ValueError, match='Email cannot be empty'):
            self.user_service.create_user('', 'Juan')

        self.user_repository.save.assert_not_called()
        self.email_service.send_welcome_email.assert_not_called()

    def test_get_user_not_found(self, setup):
        # Arrange
        self.user_repository.find_by_id.return_value = None

        # Act & Assert
        with pytest.raises(Exception, match='User not found'):
            self.user_service.get_user_by_id(999)

2. PRUEBAS DE INTEGRACION
Herramienta KARATE
# tests/features/user_api.feature
Feature: User API Tests

  Background:
    * url 'http://localhost:8080/api'
    * header Content-Type = 'application/json'

  Scenario: Create user successfully
    Given path '/users'
    And request { email: 'juan@example.com', name: 'Juan Pérez' }
    When method post
    Then status 201
    And match response == { id: '#number', email: 'juan@example.com', name: 'Juan Pérez' }

  Scenario: Get user by ID
    Given path '/users/1'
    When method get
    Then status 200
    And match response.email == 'juan@example.com'

  Scenario: Update user
    Given path '/users/1'
    And request { name: 'Juan Updated' }
    When method put
    Then status 200
    And match response.name == 'Juan Updated'

  Scenario: Delete user
    Given path '/users/1'
    When method delete
    Then status 204

  Scenario: User not found
    Given path '/users/999'
    When method get
    Then status 404

  Scenario: Invalid email
    Given path '/users'
    And request { email: '', name: 'Juan' }
    When method post
    Then status 400
    And match response.error == 'Email is required'

<!-- pom.xml -->
<dependency>
    <groupId>com.intuit.karate</groupId>
    <artifactId>karate-junit5</artifactId>
    <version>1.3.1</version>
    <scope>test</scope>
</dependency>

// src/test/java/UserApiTest.java
import com.intuit.karate.junit5.Karate;

class UserApiTest {
    @Karate.Test
    Karate testAll() {
        return Karate.run("classpath:features").relativeTo(getClass());
    }
}

Ejecutar:
mvn test
# O específico
mvn test -Dtest=UserApiTest

3. PRUEBAS E2E CON SERENITY
ARQUITECTURA PROPUESTA
tests/
├── karate/                          # Pruebas de API (integración)
│   ├── users.feature
│   └── products.feature
│
└── serenity/                        # E2E del backend (flujos completos)
    ├── features/
    │   ├── user_registration.feature
    │   ├── user_authentication.feature
    │   ├── order_workflow.feature
    │   └── payment_processing.feature
    ├── steps/
    │   ├── UserRegistrationSteps.java
    │   ├── AuthenticationSteps.java
    │   ├── OrderSteps.java
    │   └── PaymentSteps.java
    ├── config/
    │   └── serenity.properties
    └── runners/
        └── CucumberTestRunner.java

SETUP INICIAL - SERENITY
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.automation</groupId>
    <artifactId>backend-e2e-tests</artifactId>
    <version>1.0.0</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <serenity.version>3.6.0</serenity.version>
        <serenity.cucumber.version>3.6.0</serenity.cucumber.version>
    </properties>

    <dependencies>
        <!-- Serenity BDD -->
        <dependency>
            <groupId>net.serenity-bdd</groupId>
            <artifactId>serenity-core</artifactId>
            <version>${serenity.version}</version>
        </dependency>
        
        <dependency>
            <groupId>net.serenity-bdd</groupId>
            <artifactId>serenity-cucumber</artifactId>
            <version>${serenity.cucumber.version}</version>
        </dependency>

        <!-- REST Assured para APIs -->
        <dependency>
            <groupId>io.rest-assured</groupId>
            <artifactId>rest-assured</artifactId>
            <version>5.3.1</version>
        </dependency>

        <!-- Cucumber -->
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-junit-platform-engine</artifactId>
            <version>7.12.1</version>
        </dependency>

        <!-- JUnit 5 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.9.2</version>
        </dependency>

        <!-- Assertions -->
        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <version>3.24.1</version>
        </dependency>

        <!-- JSON processing -->
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.10.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <includes>
                        <include>**/*Test.java</include>
                        <include>**/*Tests.java</include>
                    </includes>
                </configuration>
            </plugin>

            <!-- Serenity reporting -->
            <plugin>
                <groupId>net.serenity-bdd.maven.plugins</groupId>
                <artifactId>serenity-maven-plugin</artifactId>
                <version>${serenity.version}</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>aggregate</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

serenity.properties
# Configuración general
serenity.project.name = Backend E2E Tests
serenity.report.encoding = UTF-8

# Cucumber
cucumber.options = --tags @smoke --plugin json:target/cucumber.json

# Base URL
base.url = http://localhost:8080

# Browser (si necesitas UI)
webdriver.driver = chrome
chrome.switches = --headless,--no-sandbox

# Logging
serenity.logging.level = INFO

# Report
serenity.take.screenshots = FOR_EACH_ACTION

EJEMPLO COMPLETO: E2E USER REGISTRATION
Feature File: user_registration.feature

# src/test/resources/features/user_registration.feature
Feature: Complete User Registration E2E Flow

  Background:
    * the user is on the system
    * the database is clean

  @smoke @e2e
  Scenario: User registers successfully with all required fields
    Given the user wants to register a new account
    When the user provides valid registration details
      | email           | juan@example.com   |
      | password        | SecurePass123!     |
      | name            | Juan Pérez         |
      | phone           | +34 912345678      |
    And the user confirms the registration
    Then the system should create the user successfully
    And the registration confirmation email should be sent
    And the user should be able to login with the new credentials

  @smoke @e2e
  Scenario: User registration with invalid email
    Given the user wants to register a new account
    When the user provides registration details with invalid email
      | email           | invalid-email      |
      | password        | SecurePass123!     |
      | name            | Juan Pérez         |
    And the user attempts to confirm the registration
    Then the system should display validation error for email field
    And no user should be created in the database

  @regression @e2e
  Scenario: User registration validation - duplicate email
    Given a user already exists with email "existing@example.com"
    When a new user tries to register with the same email
      | email           | existing@example.com |
      | password        | SecurePass123!       |
      | name            | Another User         |
    And the user attempts to confirm the registration
    Then the system should display error "Email already registered"
    And only one user should exist with that email

  @regression @e2e
  Scenario: Complete registration workflow - activation link
    Given the user has registered with email "newuser@example.com"
    When the registration confirmation email is sent
    And the user clicks the activation link in the email
    Then the user account should be activated
    And the user should be able to login immediately

  @regression @e2e
  Scenario: Registration timeout - incomplete flow
    Given the user starts the registration process
    When the user fills in the registration form
    And waits for 31 minutes without completing registration
    Then the session should expire
    And the registration form data should be cleared
    And the user should be redirected to start over

Steps: UserRegistrationSteps.java

// src/test/java/com/automation/steps/UserRegistrationSteps.java
package com.automation.steps;

import io.cucumber.java.Before;
import io.cucumber.java.After;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.When;
import io.cucumber.java.en.And;
import io.cucumber.java.en.Then;
import io.cucumber.datatable.DataTable;

import net.serenitybdd.core.pages.PageObject;
import net.serenitybdd.rest.SerenityRest;
import net.serenitybdd.screenplay.Actor;
import net.serenitybdd.screenplay.abilities.BrowseTheWeb;

import io.restassured.response.Response;
import static io.restassured.RestAssured.*;
import static org.assertj.core.api.Assertions.*;

import java.util.HashMap;
import java.util.Map;

public class UserRegistrationSteps {

    private String baseUrl = "http://localhost:8080/api";
    private Response lastResponse;
    private Map<String, String> registrationData = new HashMap<>();
    private String createdUserId;
    private String activationToken;

    // ============ BACKGROUND / SETUP ============

    @Before
    public void setUp() {
        registrationData.clear();
        createdUserId = null;
        activationToken = null;
    }

    @After
    public void tearDown() {
        // Cleanup si es necesario
    }

    // ============ GIVEN ============

    @Given("the user is on the system")
    public void userIsOnTheSystem() {
        // Verificar que el sistema está disponible
        lastResponse = given()
            .when()
            .get(baseUrl + "/health")
            .then()
            .statusCode(200)
            .extract()
            .response();
    }

    @Given("the database is clean")
    public void databaseIsClean() {
        // Cleanup de datos de prueba previos
        given()
            .when()
            .delete(baseUrl + "/test/cleanup")
            .then()
            .statusCode(204);
    }

    @Given("the user wants to register a new account")
    public void userWantsToRegister() {
        // Solo inicializar el estado
        registrationData.clear();
    }

    @Given("a user already exists with email {string}")
    public void userAlreadyExists(String email) {
        // Crear un usuario en la BD
        Map<String, String> existingUser = new HashMap<>();
        existingUser.put("email", email);
        existingUser.put("password", "ExistingPass123!");
        existingUser.put("name", "Existing User");

        lastResponse = given()
            .contentType("application/json")
            .body(existingUser)
            .when()
            .post(baseUrl + "/auth/register")
            .then()
            .statusCode(201)
            .extract()
            .response();

        createdUserId = lastResponse.jsonPath().getString("id");
    }

    @Given("the user has registered with email {string}")
    public void userHasRegisteredWithEmail(String email) {
        // Realizar el registro
        Map<String, String> userData = new HashMap<>();
        userData.put("email", email);
        userData.put("password", "SecurePass123!");
        userData.put("name", "New User");

        lastResponse = given()
            .contentType("application/json")
            .body(userData)
            .when()
            .post(baseUrl + "/auth/register")
            .then()
            .statusCode(201)
            .extract()
            .response();

        createdUserId = lastResponse.jsonPath().getString("id");
        
        // Capturar el token de activación (normalmente vendría en el email)
        activationToken = lastResponse.jsonPath().getString("activationToken");
    }

    @Given("the user starts the registration process")
    public void userStartsRegistration() {
        // Simular inicio de sesión en el registro
        registrationData.clear();
    }

    // ============ WHEN ============

    @When("the user provides valid registration details")
    public void userProvidesValidDetails(DataTable dataTable) {
        // Mapear los datos de la tabla
        Map<String, String> details = dataTable.asMap(String.class, String.class);
        registrationData.putAll(details);
    }

    @When("the user provides registration details with invalid email")
    public void userProvidesInvalidEmailDetails(DataTable dataTable) {
        Map<String, String> details = dataTable.asMap(String.class, String.class);
        registrationData.putAll(details);
    }

    @When("the user confirms the registration")
    public void userConfirmsRegistration() {
        // Realizar POST a registro
        lastResponse = given()
            .contentType("application/json")
            .body(registrationData)
            .when()
            .post(baseUrl + "/auth/register")
            .then()
            .extract()
            .response();

        if (lastResponse.statusCode() == 201) {
            createdUserId = lastResponse.jsonPath().getString("id");
            activationToken = lastResponse.jsonPath().getString("activationToken");
        }
    }

    @When("the user attempts to confirm the registration")
    public void userAttemptsConfirmation() {
        lastResponse = given()
            .contentType("application/json")
            .body(registrationData)
            .when()
            .post(baseUrl + "/auth/register")
            .then()
            .extract()
            .response();
    }

    @When("a new user tries to register with the same email")
    public void newUserTriesRegisterWithSameEmail(DataTable dataTable) {
        Map<String, String> details = dataTable.asMap(String.class, String.class);
        registrationData.putAll(details);
    }

    @When("the registration confirmation email is sent")
    public void registrationEmailSent() {
        // Verificar que el email fue enviado
        lastResponse = given()
            .when()
            .get(baseUrl + "/test/emails?userId=" + createdUserId)
            .then()
            .extract()
            .response();

        assertThat(lastResponse.statusCode()).isEqualTo(200);
        assertThat(lastResponse.jsonPath().getList("", Object.class)).isNotEmpty();
    }

    @When("the user clicks the activation link in the email")
    public void userClicksActivationLink() {
        // Simular click en el link (usar el token)
        lastResponse = given()
            .param("token", activationToken)
            .when()
            .post(baseUrl + "/auth/activate")
            .then()
            .extract()
            .response();
    }

    @When("the user fills in the registration form")
    public void userFillsRegistrationForm() {
        registrationData.put("email", "tempuser@example.com");
        registrationData.put("password", "SecurePass123!");
        registrationData.put("name", "Temp User");
    }

    @When("waits for {int} minutes without completing registration")
    public void waitsWithoutCompletion(int minutes) {
        // Simular espera
        try {
            Thread.sleep(minutes * 60 * 1000); // milliseconds
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    // ============ AND ============

    @And("the system should create the user successfully")
    public void systemCreatesUserSuccessfully() {
        assertThat(lastResponse.statusCode())
            .isEqualTo(201);
        
        assertThat(lastResponse.jsonPath().getString("id"))
            .isNotNull();
        
        assertThat(lastResponse.jsonPath().getString("email"))
            .isEqualTo(registrationData.get("email"));
    }

    @And("the registration confirmation email should be sent")
    public void registrationEmailShouldBeSent() {
        // Verificar que el email fue enviado
        lastResponse = given()
            .when()
            .get(baseUrl + "/test/emails?userId=" + createdUserId + "&type=activation")
            .then()
            .extract()
            .response();

        assertThat(lastResponse.statusCode()).isEqualTo(200);
        assertThat(lastResponse.jsonPath().getList("", Object.class)).hasSize(1);
    }

    @And("the user should be able to login with the new credentials")
    public void userShouldBeAbleToLogin() {
        Map<String, String> loginData = new HashMap<>();
        loginData.put("email", registrationData.get("email"));
        loginData.put("password", registrationData.get("password"));

        lastResponse = given()
            .contentType("application/json")
            .body(loginData)
            .when()
            .post(baseUrl + "/auth/login")
            .then()
            .statusCode(200)
            .extract()
            .response();

        assertThat(lastResponse.jsonPath().getString("token"))
            .isNotNull();
    }

    @And("no user should be created in the database")
    public void noUserShouldBeCreated() {
        // Verificar que el usuario NO existe
        lastResponse = given()
            .when()
            .get(baseUrl + "/users?email=" + registrationData.get("email"))
            .then()
            .extract()
            .response();

        assertThat(lastResponse.jsonPath().getList("", Object.class))
            .isEmpty();
    }

    @Then("the system should display validation error for email field")
    public void systemDisplaysEmailValidationError() {
        assertThat(lastResponse.statusCode())
            .isEqualTo(400);
        
        String errorMessage = lastResponse.jsonPath().getString("errors[0].field");
        assertThat(errorMessage).isEqualTo("email");
    }

    @Then("the system should display error {string}")
    public void systemDisplaysError(String expectedError) {
        assertThat(lastResponse.statusCode())
            .isEqualTo(409); // Conflict
        
        String error = lastResponse.jsonPath().getString("error");
        assertThat(error).contains(expectedError);
    }

    @And("only one user should exist with that email")
    public void onlyOneUserWithEmail() {
        String email = registrationData.get("email");
        
        lastResponse = given()
            .when()
            .get(baseUrl + "/users?email=" + email)
            .then()
            .extract()
            .response();

        assertThat(lastResponse.jsonPath().getList("", Object.class))
            .hasSize(1);
    }

    @Then("the user account should be activated")
    public void userAccountShouldBeActivated() {
        assertThat(lastResponse.statusCode()).isEqualTo(200);
        
        // Verificar en BD que el usuario está activo
        lastResponse = given()
            .when()
            .get(baseUrl + "/users/" + createdUserId)
            .then()
            .extract()
            .response();

        String isActive = lastResponse.jsonPath().getString("isActive");
        assertThat(isActive).isEqualTo("true");
    }

    @And("the user should be able to login immediately")
    public void userCanLoginImmediately() {
        // Ya verificado en pasos anteriores
        assertThat(lastResponse.statusCode()).isEqualTo(200);
    }

    @Then("the session should expire")
    public void sessionShouldExpire() {
        assertThat(lastResponse.statusCode())
            .isIn(401, 403); // Unauthorized o Forbidden
    }

    @And("the registration form data should be cleared")
    public void registrationFormDataCleared() {
        registrationData.clear();
        assertThat(registrationData).isEmpty();
    }

    @And("the user should be redirected to start over")
    public void userRedirectedToStartOver() {
        // Verificar redirect
        assertThat(lastResponse.getStatusCode())
            .isEqualTo(302); // Redirect
    }
}

Test Runner: CucumberTestRunner.java
// src/test/java/com/automation/runners/CucumberTestRunner.java
package com.automation.runners;

import io.cucumber.junit.platform.engine.Constants;
import org.junit.platform.suite.api.ConfigurationParameter;
import org.junit.platform.suite.api.SelectClasspathResource;
import org.junit.platform.suite.api.Suite;

@Suite
@SelectClasspathResource("features")
@ConfigurationParameter(
    key = Constants.PLUGIN_PROPERTY_NAME,
    value = "pretty, json:target/cucumber-reports/cucumber.json"
)
@ConfigurationParameter(
    key = Constants.GLUE_PROPERTY_NAME,
    value = "com.automation.steps"
)
public class CucumberTestRunner {
    // Test runner for Cucumber/Serenity
}

PRUEBAS UNITARIAS - REACT CON JEST
Setup Inicial
# Crear proyecto React
npx create-react-app mi-app
cd mi-app

# O con Vite (más rápido)
npm create vite@latest mi-app -- --template react
cd mi-app

# Dependencias de testing ya vienen con React
# Si no, instalar:
npm install --save-dev @testing-library/react @testing-library/jest-dom @testing-library/user-event

package.json
{
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "axios": "^1.4.0"
  },
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^5.16.5",
    "@testing-library/user-event": "^14.4.3",
    "jest": "^29.5.0",
    "@babel/preset-react": "^7.22.0"
  },
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}

jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/src/setupTests.js'],
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/index.js',
    '!src/reportWebVitals.js',
  ],
};

src/setupTests.js
import '@testing-library/jest-dom';

COMPONENTES REACT
Componente 1: UserRegistration.jsx
// src/components/UserRegistration.jsx
import React, { useState } from 'react';
import axios from 'axios';

export const UserRegistration = () => {
  const [formData, setFormData] = useState({
    email: '',
    password: '',
    name: '',
  });
  const [errors, setErrors] = useState({});
  const [success, setSuccess] = useState(false);
  const [loading, setLoading] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: value,
    }));
    // Limpiar error del campo cuando el usuario empieza a escribir
    if (errors[name]) {
      setErrors((prev) => ({
        ...prev,
        [name]: '',
      }));
    }
  };

  const validateForm = () => {
    const newErrors = {};

    if (!formData.email) {
      newErrors.email = 'Email is required';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    if (!formData.name) {
      newErrors.name = 'Name is required';
    }

    return newErrors;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    const newErrors = validateForm();
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    setLoading(true);
    try {
      await axios.post('http://localhost:8080/api/auth/register', formData);
      setSuccess(true);
      setFormData({ email: '', password: '', name: '' });
      setErrors({});
    } catch (error) {
      if (error.response?.data?.error) {
        setErrors({ form: error.response.data.error });
      } else {
        setErrors({ form: 'Registration failed. Please try again.' });
      }
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="registration-container">
      <h2>User Registration</h2>

      {success && (
        <div className="success-message" data-testid="success-message">
          Registration successful! You can now login.
        </div>
      )}

      {errors.form && (
        <div className="error-message" data-testid="form-error">
          {errors.form}
        </div>
      )}

      <form onSubmit={handleSubmit} data-testid="registration-form">
        <div className="form-group">
          <label htmlFor="email">Email:</label>
          <input
            id="email"
            type="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
            data-testid="email-input"
          />
          {errors.email && (
            <span className="field-error" data-testid="email-error">
              {errors.email}
            </span>
          )}
        </div>

        <div className="form-group">
          <label htmlFor="password">Password:</label>
          <input
            id="password"
            type="password"
            name="password"
            value={formData.password}
            onChange={handleChange}
            data-testid="password-input"
          />
          {errors.password && (
            <span className="field-error" data-testid="password-error">
              {errors.password}
            </span>
          )}
        </div>

        <div className="form-group">
          <label htmlFor="name">Name:</label>
          <input
            id="name"
            type="text"
            name="name"
            value={formData.name}
            onChange={handleChange}
            data-testid="name-input"
          />
          {errors.name && (
            <span className="field-error" data-testid="name-error">
              {errors.name}
            </span>
          )}
        </div>

        <button type="submit" disabled={loading} data-testid="submit-button">
          {loading ? 'Registering...' : 'Register'}
        </button>
      </form>
    </div>
  );
};

Pruebas Unitarias: UserRegistration.test.jsx
// src/components/UserRegistration.test.jsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import axios from 'axios';
import { UserRegistration } from './UserRegistration';

// Mock de axios
jest.mock('axios');

describe('UserRegistration Component', () => {
  afterEach(() => {
    jest.clearAllMocks();
  });

  // ============ TESTS DE RENDERIZADO ============

  test('should render registration form', () => {
    render(<UserRegistration />);

    expect(screen.getByText('User Registration')).toBeInTheDocument();
    expect(screen.getByTestId('registration-form')).toBeInTheDocument();
    expect(screen.getByTestId('email-input')).toBeInTheDocument();
    expect(screen.getByTestId('password-input')).toBeInTheDocument();
    expect(screen.getByTestId('name-input')).toBeInTheDocument();
  });

  test('should render submit button', () => {
    render(<UserRegistration />);

    const submitButton = screen.getByTestId('submit-button');
    expect(submitButton).toBeInTheDocument();
    expect(submitButton).toHaveTextContent('Register');
  });

  // ============ TESTS DE VALIDACIÓN ============

  test('should show error when email is empty', async () => {
    render(<UserRegistration />);

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByTestId('email-error')).toBeInTheDocument();
      expect(screen.getByTestId('email-error')).toHaveTextContent(
        'Email is required'
      );
    });
  });

  test('should show error for invalid email format', async () => {
    render(<UserRegistration />);

    const emailInput = screen.getByTestId('email-input');
    await userEvent.type(emailInput, 'invalid-email');

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByTestId('email-error')).toHaveTextContent(
        'Invalid email format'
      );
    });
  });

  test('should show error when password is empty', async () => {
    render(<UserRegistration />);

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByTestId('password-error')).toBeInTheDocument();
      expect(screen.getByTestId('password-error')).toHaveTextContent(
        'Password is required'
      );
    });
  });

  test('should show error when password is less than 8 characters', async () => {
    render(<UserRegistration />);

    const passwordInput = screen.getByTestId('password-input');
    await userEvent.type(passwordInput, 'short');

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByTestId('password-error')).toHaveTextContent(
        'Password must be at least 8 characters'
      );
    });
  });

  test('should show error when name is empty', async () => {
    render(<UserRegistration />);

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByTestId('name-error')).toBeInTheDocument();
      expect(screen.getByTestId('name-error')).toHaveTextContent(
        'Name is required'
      );
    });
  });

  test('should show multiple errors at once', async () => {
    render(<UserRegistration />);

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByTestId('email-error')).toBeInTheDocument();
      expect(screen.getByTestId('password-error')).toBeInTheDocument();
      expect(screen.getByTestId('name-error')).toBeInTheDocument();
    });
  });

  // ============ TESTS DE ENVÍO DEL FORMULARIO ============

  test('should register user successfully with valid data', async () => {
    axios.post.mockResolvedValue({
      data: { id: 1, email: 'juan@example.com', name: 'Juan Pérez' },
    });

    render(<UserRegistration />);

    const emailInput = screen.getByTestId('email-input');
    const passwordInput = screen.getByTestId('password-input');
    const nameInput = screen.getByTestId('name-input');

    await userEvent.type(emailInput, 'juan@example.com');
    await userEvent.type(passwordInput, 'SecurePass123');
    await userEvent.type(nameInput, 'Juan Pérez');

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(axios.post).toHaveBeenCalledWith(
        'http://localhost:8080/api/auth/register',
        {
          email: 'juan@example.com',
          password: 'SecurePass123',
          name: 'Juan Pérez',
        }
      );
    });

    expect(screen.getByTestId('success-message')).toBeInTheDocument();
  });

  test('should show loading state while registering', async () => {
    // Mock una promesa que tarda
    axios.post.mockImplementation(
      () =>
        new Promise((resolve) =>
          setTimeout(() => resolve({ data: { id: 1 } }), 100)
        )
    );

    render(<UserRegistration />);

    await userEvent.type(screen.getByTestId('email-input'), 'juan@example.com');
    await userEvent.type(screen.getByTestId('password-input'), 'SecurePass123');
    await userEvent.type(screen.getByTestId('name-input'), 'Juan Pérez');

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    // Button debe mostrar "Registering..." y estar deshabilitado
    expect(submitButton).toHaveTextContent('Registering...');
    expect(submitButton).toBeDisabled();

    await waitFor(() => {
      expect(submitButton).toHaveTextContent('Register');
      expect(submitButton).not.toBeDisabled();
    });
  });

  test('should handle registration error', async () => {
    axios.post.mockRejectedValue({
      response: {
        data: { error: 'Email already exists' },
      },
    });

    render(<UserRegistration />);

    await userEvent.type(screen.getByTestId('email-input'), 'existing@example.com');
    await userEvent.type(screen.getByTestId('password-input'), 'SecurePass123');
    await userEvent.type(screen.getByTestId('name-input'), 'User');

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByTestId('form-error')).toHaveTextContent(
        'Email already exists'
      );
    });
  });

  test('should clear errors when user starts typing', async () => {
    render(<UserRegistration />);

    // Trigger error
    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(screen.getByTestId('email-error')).toBeInTheDocument();
    });

    // Escribir en email
    const emailInput = screen.getByTestId('email-input');
    await userEvent.type(emailInput, 'juan@example.com');

    // El error debe desaparecer
    expect(screen.queryByTestId('email-error')).not.toBeInTheDocument();
  });

  test('should reset form after successful registration', async () => {
    axios.post.mockResolvedValue({ data: { id: 1 } });

    render(<UserRegistration />);

    const emailInput = screen.getByTestId('email-input');
    const passwordInput = screen.getByTestId('password-input');
    const nameInput = screen.getByTestId('name-input');

    await userEvent.type(emailInput, 'juan@example.com');
    await userEvent.type(passwordInput, 'SecurePass123');
    await userEvent.type(nameInput, 'Juan Pérez');

    const submitButton = screen.getByTestId('submit-button');
    fireEvent.click(submitButton);

    await waitFor(() => {
      expect(emailInput).toHaveValue('');
      expect(passwordInput).toHaveValue('');
      expect(nameInput).toHaveValue('');
    });
  });
});

EJECUTAR PRUEBAS UNITARIAS CON JEST
# Todos los tests
npm test

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage

# Test específico
npm test UserRegistration.test.jsx

# Tests de un archivo
npm test -- --testPathPattern="UserRegistration"

PRUEBAS E2E - PLAYWRIGHT

Setup Playwright
# Instalar Playwright
npm install --save-dev @playwright/test

# Inicializar configuración
npx playwright install
npx playwright codegen http://localhost:3000 # Para generar tests automáticamente

playwright.config.ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,

  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results/results.json' }],
    ['junit', { outputFile: 'test-results/results.xml' }],
  ],

  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});

Test 1: Prueba de Registro E2E
// e2e/user-registration.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Registration E2E', () => {
  
  test.beforeEach(async ({ page }) => {
    // Navegar a la página
    await page.goto('/register');
  });

  test('should register a new user successfully', async ({ page }) => {
    // Arrange - Datos de prueba
    const testEmail = `user-${Date.now()}@example.com`;
    const testPassword = 'SecurePassword123';
    const testName = 'Juan Test';

    // Act - Rellenar formulario
    await page.fill('input[data-testid="email-input"]', testEmail);
    await page.fill('input[data-testid="password-input"]', testPassword);
    await page.fill('input[data-testid="name-input"]', testName);

    // Enviar formulario
    await page.click('button[data-testid="submit-button"]');

    // Assert - Verificar éxito
    const successMessage = page.locator('[data-testid="success-message"]');
    await expect(successMessage).toBeVisible();
    await expect(successMessage).toContainText('Registration successful');
  });

  test('should show error for invalid email', async ({ page }) => {
    // Act
    await page.fill('input[data-testid="email-input"]', 'invalid-email');
    await page.fill('input[data-testid="password-input"]', 'SecurePass123');
    await page.fill('input[data-testid="name-input"]', 'Test User');
    await page.click('button[data-testid="submit-button"]');

    // Assert
    const emailError = page.locator('[data-testid="email-error"]');
    await expect(emailError).toBeVisible();
    await expect(emailError).toContainText('Invalid email format');
  });

  test('should show error for short password', async ({ page }) => {
    // Act
    await page.fill('input[data-testid="email-input"]', 'test@example.com');
    await page.fill('input[data-testid="password-input"]', 'short');
    await page.fill('input[data-testid="name-input"]', 'Test User');
    await page.click('button[data-testid="submit-button"]');

    // Assert
    const passwordError = page.locator('[data-testid="password-error"]');
    await expect(passwordError).toBeVisible();
    await expect(passwordError).toContainText('at least 8 characters');
  });

  test('should show error for empty email', async ({ page }) => {
    // Act
    await page.fill('input[data-testid="password-input"]', 'SecurePass123');
    await page.fill('input[data-testid="name-input"]', 'Test User');
    await page.click('button[data-testid="submit-button"]');

    // Assert
    const emailError = page.locator('[data-testid="email-error"]');
    await expect(emailError).toBeVisible();
    await expect(emailError).toContainText('Email is required');
  });

  test('should show all errors at once', async ({ page }) => {
    // Act - Enviar form vacío
    await page.click('button[data-testid="submit-button"]');

    // Assert
    const emailError = page.locator('[data-testid="email-error"]');
    const passwordError = page.locator('[data-testid="password-error"]');
    const nameError = page.locator('[data-testid="name-error"]');

    await expect(emailError).toBeVisible();
    await expect(passwordError).toBeVisible();
    await expect(nameError).toBeVisible();
  });

  test('should show loading state during submission', async ({ page }) => {
    // Act - Interceptar la solicitud para que tarde
    await page.route('**/api/auth/register', (route) => {
      setTimeout(() => route.continue(), 1000);
    });

    await page.fill('input[data-testid="email-input"]', 'test@example.com');
    await page.fill('input[data-testid="password-input"]', 'SecurePass123');
    await page.fill('input[data-testid="name-input"]', 'Test User');

    const submitButton = page.locator('button[data-testid="submit-button"]');
    await submitButton.click();

    // Assert - Verificar que muestra "Registering..." y está deshabilitado
    await expect(submitButton).toContainText('Registering...');
    await expect(submitButton).toBeDisabled();
  });

  test('should clear errors when user starts typing', async ({ page }) => {
    // Trigger error
    await page.click('button[data-testid="submit-button"]');

    let emailError = page.locator('[data-testid="email-error"]');
    await expect(emailError).toBeVisible();

    // Escribir en el campo
    await page.fill('input[data-testid="email-input"]', 'test@example.com');

    // El error debe desaparecer
    await expect(emailError).not.toBeVisible();
  });

  test('should handle duplicate email error', async ({ page }) => {
    // Mock API para simular email duplicado
    await page.route('**/api/auth/register', (route) => {
      route.abort();
    });

    // Act
    await page.fill('input[data-testid="email-input"]', 'existing@example.com');
    await page.fill('input[data-testid="password-input"]', 'SecurePass123');
    await page.fill('input[data-testid="name-input"]', 'Test User');
    await page.click('button[data-testid="submit-button"]');

    // Assert
    const formError = page.locator('[data-testid="form-error"]');
    await expect(formError).toBeVisible();
  });
});

EJECUTAR PRUEBAS CON PLAYWRIGTH
# Ejecutar todos los tests
npx playwright test

# Watch mode
npx playwright test --watch

# Modo UI
npx playwright test --ui

# Específico navegador
npx playwright test --project=chromium

# Específico test
npx playwright test user-registration.spec.ts

# Con reporte HTML
npx playwright test --reporter=html
open playwright-report/index.html

# Con traza (debugging)
npx playwright test --trace=on