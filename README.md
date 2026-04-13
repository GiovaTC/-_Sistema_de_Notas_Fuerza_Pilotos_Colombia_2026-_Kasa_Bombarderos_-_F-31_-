# -_Sistema_de_Notas_Fuerza_Pilotos_Colombia_2026-_Kasa_Bombarderos_-_F-31_- :.
✈️ Sistema de Notas:

<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/6a35ff00-1a03-4d78-8f36-c6b825d77ee1" />  

```

Fuerza Pilotos Colombia 2026 – Kasa Bombarderos F-31

Solución completa, funcional y estructurada en Java (IntelliJ):

✅ Características
Interfaz gráfica (Swing)
Tabla temporal (notas por asignatura)
Confirmación de registros
Tabla confirmada (resultado final: aprobado/reprobado)
Manejo de 12 estudiantes
Arquitectura (Modelo + DAO + Service + Vista + Controlador).

🔗 Conexión real a Oracle 19c (JDBC)
⚠️ 0. Dependencia Maven
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>19.3.0.0</version>
</dependency>

🗄️ 1. Script Base de Datos (Oracle 19c)
CREATE TABLE ESTUDIANTES_CONFIRMADOS (
    ID NUMBER GENERATED ALWAYS AS IDENTITY,
    NOMBRE VARCHAR2(100),
    PROMEDIO NUMBER(4,2),
    ESTADO VARCHAR2(20)
);

🧩 2. Estructura del Proyecto
src/
 ├── model/
 │    ├── EstudianteTemp.java
 │    └── EstudianteConfirmado.java
 │
 ├── dao/
 │    ├── ConexionBD.java
 │    └── EstudianteDAO.java
 │
 ├── service/
 │    └── EstudianteService.java
 │
 ├── view/
 │    └── Vista.java
 │
 ├── controller/
 │    └── Controlador.java
 │
 └── Main.java

🧱 3. Modelos
📌 EstudianteTemp.java
package model;

public class EstudianteTemp {
    private String nombre;
    private double nota1, nota2, nota3, nota4, nota5;

    public EstudianteTemp(String nombre, double n1, double n2, double n3, double n4, double n5) {
        this.nombre = nombre;
        this.nota1 = n1;
        this.nota2 = n2;
        this.nota3 = n3;
        this.nota4 = n4;
        this.nota5 = n5;
    }

    public String getNombre() { return nombre; }

    public double promedio() {
        return (nota1 + nota2 + nota3 + nota4 + nota5) / 5;
    }
}

📌 EstudianteConfirmado.java
package model;

public class EstudianteConfirmado {
    private String nombre;
    private double promedio;
    private String estado;

    public EstudianteConfirmado(String nombre, double promedio, String estado) {
        this.nombre = nombre;
        this.promedio = promedio;
        this.estado = estado;
    }

    public String getNombre() { return nombre; }
    public double getPromedio() { return promedio; }
    public String getEstado() { return estado; }
}

🔌 4. Conexión a Oracle
📌 ConexionBD.java
package dao;

import java.sql.Connection;
import java.sql.DriverManager;

public class ConexionBD {

    private static final String URL = "jdbc:oracle:thin:@localhost:1521:xe";
    private static final String USER = "system";
    private static final String PASS = "123456";

    public static Connection getConexion() throws Exception {
        return DriverManager.getConnection(URL, USER, PASS);
    }
}

🗄️ 5. DAO (Con JDBC)
📌 EstudianteDAO.java
package dao;

import model.*;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.util.*;

public class EstudianteDAO {

    private List<EstudianteTemp> tablaTemporal = new ArrayList<>();
    private List<EstudianteConfirmado> tablaConfirmada = new ArrayList<>();

    public void guardarTemporal(EstudianteTemp e) {
        tablaTemporal.add(e);
    }

    public List<EstudianteTemp> obtenerTemporales() {
        return tablaTemporal;
    }

    public void confirmar() {
        tablaConfirmada.clear();

        for (EstudianteTemp e : tablaTemporal) {
            double promedio = e.promedio();
            String estado = promedio >= 3.0 ? "APROBADO" : "REPROBADO";

            EstudianteConfirmado ec =
                    new EstudianteConfirmado(e.getNombre(), promedio, estado);

            tablaConfirmada.add(ec);

            insertarEnBD(ec);
        }
    }

    private void insertarEnBD(EstudianteConfirmado e) {
        String sql = "INSERT INTO ESTUDIANTES_CONFIRMADOS (NOMBRE, PROMEDIO, ESTADO) VALUES (?, ?, ?)";

        try (Connection conn = ConexionBD.getConexion();
             PreparedStatement ps = conn.prepareStatement(sql)) {

            ps.setString(1, e.getNombre());
            ps.setDouble(2, e.getPromedio());
            ps.setString(3, e.getEstado());

            ps.executeUpdate();

        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }

    public List<EstudianteConfirmado> obtenerConfirmados() {
        return tablaConfirmada;
    }
}

⚙️ 6. Service
📌 EstudianteService.java
package service;

import dao.EstudianteDAO;
import model.*;
import java.util.List;

public class EstudianteService {

    private EstudianteDAO dao = new EstudianteDAO();

    public void cargarDatosEjemplo() {
        for (int i = 1; i <= 12; i++) {
            dao.guardarTemporal(new EstudianteTemp(
                "Piloto F-31 #" + i,
                Math.random()*5,
                Math.random()*5,
                Math.random()*5,
                Math.random()*5,
                Math.random()*5
            ));
        }
    }

    public List<EstudianteTemp> getTemporales() {
        return dao.obtenerTemporales();
    }

    public void confirmar() {
        dao.confirmar();
    }

    public List<EstudianteConfirmado> getConfirmados() {
        return dao.obtenerConfirmados();
    }
}

🖥️ 7. Vista (Swing)
📌 Vista.java
package view;

import javax.swing.*;
import javax.swing.table.DefaultTableModel;
import java.awt.*;

public class Vista extends JFrame {

    public JTable tablaTemp;
    public JTable tablaConfirmada;
    public JButton btnCargar, btnConfirmar;

    public Vista() {
        setTitle("Fuerza Pilotos Colombia 2026 - Kasa F-31");
        setSize(900, 600);
        setDefaultCloseOperation(EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        tablaTemp = new JTable();
        tablaConfirmada = new JTable();

        btnCargar = new JButton("Cargar 12 Pilotos");
        btnConfirmar = new JButton("Confirmar Notas");

        JPanel panelBotones = new JPanel();
        panelBotones.add(btnCargar);
        panelBotones.add(btnConfirmar);

        add(panelBotones, BorderLayout.NORTH);

        JSplitPane split = new JSplitPane(
                JSplitPane.VERTICAL_SPLIT,
                new JScrollPane(tablaTemp),
                new JScrollPane(tablaConfirmada)
        );

        add(split, BorderLayout.CENTER);
    }

    public void mostrarTemporales(Object[][] data) {
        tablaTemp.setModel(new DefaultTableModel(
                data,
                new String[]{"Nombre", "Promedio"}
        ));
    }

    public void mostrarConfirmados(Object[][] data) {
        tablaConfirmada.setModel(new DefaultTableModel(
                data,
                new String[]{"Nombre", "Promedio", "Estado"}
        ));
    }
}

🎮 8. Controlador
📌 Controlador.java
package controller;

import view.Vista;
import service.EstudianteService;
import model.*;

import java.awt.event.ActionEvent;
import java.util.List;

public class Controlador {

    private Vista vista;
    private EstudianteService service;

    public Controlador(Vista vista) {
        this.vista = vista;
        this.service = new EstudianteService();

        vista.btnCargar.addActionListener(this::cargar);
        vista.btnConfirmar.addActionListener(this::confirmar);
    }

    private void cargar(ActionEvent e) {
        service.cargarDatosEjemplo();

        List<EstudianteTemp> lista = service.getTemporales();
        Object[][] data = new Object[lista.size()][2];

        for (int i = 0; i < lista.size(); i++) {
            data[i][0] = lista.get(i).getNombre();
            data[i][1] = lista.get(i).promedio();
        }

        vista.mostrarTemporales(data);
    }

    private void confirmar(ActionEvent e) {
        service.confirmar();

        List<EstudianteConfirmado> lista = service.getConfirmados();
        Object[][] data = new Object[lista.size()][3];

        for (int i = 0; i < lista.size(); i++) {
            data[i][0] = lista.get(i).getNombre();
            data[i][1] = lista.get(i).getPromedio();
            data[i][2] = lista.get(i).getEstado();
        }

        vista.mostrarConfirmados(data);
    }
}

🚀 9. Main
📌 Main.java
import view.Vista;
import controller.Controlador;

public class Main {
    public static void main(String[] args) {
        Vista vista = new Vista();
        new Controlador(vista);
        vista.setVisible(true);
    }
}

🎯 Resultado Final
🔹 Flujo del Sistema
Cargar 12 Pilotos
Genera datos aleatorios
Muestra tabla temporal
Confirmar Notas
Calcula promedio
Determina estado.

🔗 Inserta en Oracle 19C:
Muestra tabla confirmada
🧠 Mejoras Recomendadas
🔁 Batch insert (mejor rendimiento)
🔗 Stored Procedures
📥 Lectura desde Oracle
🏗️ Pool de conexiones (HikariCP)
📊 Dashboard con gráficos :. . / .
