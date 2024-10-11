LOGIN FLUTTER

**1. En el proyecto API que tenemos en la ruta C://Xampp/htdocs/api_flutter, deberá crear el fichero de nombre "login.php"**
Escribir el comando: 

Linea::

  <?xml version="1.0" encoding="utf-8"?>
  <androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:id="@+id/main"
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      tools:context=".MainActivity">
  
      <LinearLayout
          android:id="@+id/linearLayout"
          android:layout_width="0dp"
          android:layout_height="wrap_content"
          android:orientation="vertical"
          android:padding="16dp"
          app:layout_constraintEnd_toEndOf="parent"
          app:layout_constraintStart_toStartOf="parent"
          app:layout_constraintTop_toTopOf="parent">
  
          <TextView
              android:layout_width="match_parent"
              android:textAlignment="center"
              android:layout_height="wrap_content"
              android:layout_marginBottom="16dp"
              android:text="Registro de Contacto"
              android:textSize="24sp"
              android:textStyle="bold" />
  
          <TextView
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_marginTop="8dp"
              android:text="Usuario:" />
  
          <EditText
              android:id="@+id/txtnombreUsuario"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:hint="Nombre de usuario" />
  
          <TextView
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_marginTop="8dp"
              android:text="Teléfono:" />
  
          <LinearLayout
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:orientation="horizontal">
  
              <EditText
                  android:id="@+id/txtTelefono"
                  android:layout_width="0dp"
                  android:layout_height="wrap_content"
                  android:layout_weight="1"
                  android:hint="Nro Teléfono" />
  
              <Spinner
                  android:id="@+id/spOperador"
                  android:layout_width="wrap_content"
                  android:layout_height="wrap_content"
                  android:hint="Area" />
  
          </LinearLayout>
  
          <TextView
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_marginTop="8dp"
              android:text="Dirección:" />
  
          <EditText
              android:id="@+id/txtDireccion"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:hint="Dirección" />
  
          <EditText
              android:id="@+id/txtCiudad"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:hint="Ciudad" />
  
          <LinearLayout
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:orientation="horizontal">
  
              <Spinner
                  android:id="@+id/spPais"
                  android:layout_width="0dp"
                  android:layout_height="wrap_content"
                  android:layout_weight="1"
                  android:hint="Estado" />
  
              <EditText
                  android:id="@+id/txtCodigoPostal"
                  android:layout_width="0dp"
                  android:layout_height="wrap_content"
                  android:layout_weight="1"
                  android:hint="Zip" />
  
          </LinearLayout>
  
          <TextView
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_marginTop="8dp"
              android:text="Correo:" />
  
          <EditText
              android:id="@+id/txtEmail"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:hint="Correo Electrónico" />
  
          <TextView
              android:layout_width="wrap_content"
              android:layout_height="wrap_content"
              android:layout_marginTop="8dp"
              android:text="Cumpleaños:" />
  
          <EditText
              android:id="@+id/txtFechaNacimiento"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:hint="Fecha de Nacimiento" />
  
          <Button
              android:id="@+id/btnGuardar"
              android:layout_marginTop="20dp"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:text="Guardar" />
  
  
      </LinearLayout>
  
  </androidx.constraintlayout.widget.ConstraintLayout>

**2. En el archivo LoginPage.dart; actualizar el código... que quede así**

.. image:: img/loginPage_1.png
   :height: 40
   :width: 90
   :scale: 10
   :alt: JoeAI

.. image:: img/loginPage_2.png
   :height: 40
   :width: 90
   :scale: 10
   :alt: JoeAI

.. image:: img/loginPage_3.png
   :height: 45
   :width: 90
   :scale: 10
   :alt: JoeAI

.. image:: img/loginPage_4.png
   :height: 45
   :width: 90
   :scale: 10
   :alt: JoeAI

.. image:: img/loginPage_5.png
   :height: 45
   :width: 90
   :scale: 10
   :alt: JoeAI

3. Crear el archivo **Dashboard.dart**

Instalar desde el "command Prompt", el siguiente comando:

Linea::

  flutter pub add shared_preferences

.. image:: img/Dashboard1.png
   :height: 40
   :width: 90
   :scale: 10
   :alt: JoeAI

.. image:: img/Dashboard2.png
   :height: 40
   :width: 90
   :scale: 10
   :alt: JoeAI

.. image:: img/Dashboard3.png
   :height: 45
   :width: 90
   :scale: 10
   :alt: JoeAI

3. Actualizar el archivo **main.dart**

.. image:: img/Main.dart_actualizado.png
   :height: 45
   :width: 90
   :scale: 10
   :alt: JoeAI

**Al ejecutar el archivo main.dart, deberá primero iniciar el loginPage.dart, ingresar DATOS REALES DE LA TABLA usuarios.... Al ingresar las credenciales correctas, el app les dará acceso al Dashboard.dart**
