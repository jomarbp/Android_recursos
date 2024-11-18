*CRUD COMPLETO DE TIENDA*

**1. Agregar el siguiente código "JbStoreApp\app\src\main\AndroidManifest.xml"**
Escribir el comando: 

Linea::

  <?xml version="1.0" encoding="utf-8"?>
  <manifest xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:tools="http://schemas.android.com/tools">
  
      <uses-feature
          android:name="android.hardware.camera"
          android:required="false" />
  
      <uses-permission android:name="android.permission.INTERNET" />
      <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  
      <uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>
      <uses-permission android:name="android.permission.READ_MEDIA_VISUAL_USER_SELECTED" />
  
  
  
      <application
          android:allowBackup="true"
          android:icon="@drawable/logo"
          android:label="@string/app_name"
          android:roundIcon="@drawable/logo"
          android:supportsRtl="true"
          android:theme="@style/Theme.JbStoreApp"
          android:usesCleartextTraffic="true"
          tools:targetApi="31">
          <activity
              android:name=".editar_producto"
              android:exported="false" />
          <activity
              android:name=".agregar_producto"
              android:exported="false" />
          <activity
              android:name=".lista_productos"
              android:exported="false" />
          <activity
              android:name=".editar_proveedor"
              android:exported="false" />
          <activity
              android:name=".agregar_proveedor"
              android:exported="false" />
          <activity
              android:name=".lista_proveedores"
              android:exported="false" />
          <activity
              android:name=".editar_categoria"
              android:exported="false" />
          <activity
              android:name=".agregar_categoria"
              android:exported="false" />
          <activity
              android:name=".lista_categorias"
              android:exported="false" />
          <activity
              android:name=".MainActivity"
              android:exported="true">
              <intent-filter>
                  <action android:name="android.intent.action.MAIN" />
  
                  <category android:name="android.intent.category.LAUNCHER" />
              </intent-filter>
          </activity>
      </application>
  
  </manifest>

**2. JbStoreApp\app\src\main\java\jbstoreapp\com\agregar_producto.java**

Linea::

  package jbstoreapp.com;
  
  import android.content.Intent;
  import android.content.pm.PackageManager;
  import android.graphics.Bitmap;
  import android.net.Uri;
  import android.os.Build;
  import android.os.Bundle;
  import android.provider.MediaStore;
  import android.util.Log;
  import android.view.MenuItem;
  import android.widget.ArrayAdapter;
  import android.widget.AutoCompleteTextView;
  import android.widget.Button;
  import android.widget.EditText;
  import android.widget.ImageView;
  import android.widget.Toast;
  
  import androidx.activity.result.ActivityResultLauncher;
  import androidx.activity.result.PickVisualMediaRequest;
  import androidx.activity.result.contract.ActivityResultContracts;
  import androidx.annotation.NonNull;
  import androidx.appcompat.app.AppCompatActivity;
  import androidx.appcompat.widget.SwitchCompat;
  import androidx.appcompat.widget.Toolbar;
  import androidx.core.app.ActivityCompat;
  import androidx.core.content.ContextCompat;
  
  import com.android.volley.Request;
  import com.android.volley.RequestQueue;
  import com.android.volley.toolbox.StringRequest;
  import com.android.volley.toolbox.Volley;
  import com.github.dhaval2404.imagepicker.ImagePicker;
  
  import org.json.JSONArray;
  import org.json.JSONException;
  import org.json.JSONObject;
  
  import java.io.File;
  import java.io.IOException;
  import java.util.ArrayList;
  import java.util.HashMap;
  import java.util.Map;
  import android.Manifest;
  
  import android.content.Context;
  import java.io.ByteArrayOutputStream;
  import java.io.InputStream;
  import com.android.volley.NetworkResponse;
  import com.android.volley.Response;
  import com.android.volley.VolleyError;
  
  import androidx.core.content.ContextCompat;
  
  public class agregar_producto extends AppCompatActivity {
      private EditText etNombre, etDescripcion, etPrecio, etStock;
      private AutoCompleteTextView spinnerCategoria, spinnerProveedor;
      private SwitchCompat switchEstado;
      private ImageView ivProducto;
      private Uri imageUri;
      private String URL = "http:/192.168.125.59/api_inventario/api.php";
  
      private static final int PERMISSION_REQUEST_CODE = 123;
      private static final int PICK_IMAGE_REQUEST = 1;
      private ActivityResultLauncher<PickVisualMediaRequest> pickMedia;
  
      private ArrayList<CategoriaItem> listaCategorias = new ArrayList<>();
      private ArrayList<ProveedorItem> listaProveedores = new ArrayList<>();
      private CategoriaItem categoriaSeleccionada;
      private ProveedorItem proveedorSeleccionado;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_agregar_producto);
  
          // Configurar Toolbar
          Toolbar toolbar = findViewById(R.id.toolbar);
          setSupportActionBar(toolbar);
          getSupportActionBar().setDisplayHomeAsUpEnabled(true);
          getSupportActionBar().setTitle("Agregar Producto");
  
          inicializarComponentes();
          cargarCategorias();
          cargarProveedores();
      }
  
      private void inicializarComponentes() {
          etNombre = findViewById(R.id.etNombre);
          etDescripcion = findViewById(R.id.etDescripcion);
          etPrecio = findViewById(R.id.etPrecio);
          etStock = findViewById(R.id.etStock);
          spinnerCategoria = findViewById(R.id.spinnerCategoria);
          spinnerProveedor = findViewById(R.id.spinnerProveedor);
          switchEstado = findViewById(R.id.switchEstado);
          ivProducto = findViewById(R.id.ivProducto);
  
          // Configurar Spinners
          spinnerCategoria.setOnItemClickListener((parent, view, position, id) -> {
              categoriaSeleccionada = listaCategorias.get(position);
          });
  
          spinnerProveedor.setOnItemClickListener((parent, view, position, id) -> {
              proveedorSeleccionado = listaProveedores.get(position);
          });
  
          Button btnGuardar = findViewById(R.id.btnGuardar);
          btnGuardar.setOnClickListener(v -> validarYGuardar());
  
          Button btnSeleccionarImagen = findViewById(R.id.btnSeleccionarImagen);
          btnSeleccionarImagen.setOnClickListener(v -> checkPermissionAndPickImage());
  
          // Registrar el launcher para la selección de imágenes
          pickMedia = registerForActivityResult(new ActivityResultContracts.PickVisualMedia(), uri -> {
              if (uri != null) {
                  imageUri = uri;
                  ivProducto.setImageURI(uri);
              } else {
                  Toast.makeText(this, "No se seleccionó ninguna imagen", Toast.LENGTH_SHORT).show();
              }
          });
      }
  
      private void checkPermissionAndPickImage() {
          if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
              // Android 13 y superior
              if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_MEDIA_IMAGES)
                      != PackageManager.PERMISSION_GRANTED) {
                  ActivityCompat.requestPermissions(this,
                          new String[]{Manifest.permission.READ_MEDIA_IMAGES},
                          PERMISSION_REQUEST_CODE);
              } else {
                  openImagePicker();
              }
          } else {
              // Android 12 y anterior
              if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
                      != PackageManager.PERMISSION_GRANTED) {
                  ActivityCompat.requestPermissions(this,
                          new String[]{Manifest.permission.READ_EXTERNAL_STORAGE},
                          PERMISSION_REQUEST_CODE);
              } else {
                  openImagePicker();
              }
          }
      }
  
      private void openImagePicker() {
          Intent intent = new Intent(Intent.ACTION_PICK);
          intent.setType("image/*");
          startActivityForResult(intent, PICK_IMAGE_REQUEST);
      }
  
      @Override
      public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions,
                                             @NonNull int[] grantResults) {
          super.onRequestPermissionsResult(requestCode, permissions, grantResults);
          if (requestCode == PERMISSION_REQUEST_CODE) {
              if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                  openImagePicker();
              } else {
                  Toast.makeText(this, "Permiso denegado para acceder a las imágenes",
                          Toast.LENGTH_SHORT).show();
              }
          }
      }
  
  
      @Override
      protected void onActivityResult(int requestCode, int resultCode, Intent data) {
          super.onActivityResult(requestCode, resultCode, data);
          if (requestCode == PICK_IMAGE_REQUEST && resultCode == RESULT_OK
                  && data != null && data.getData() != null) {
              imageUri = data.getData();
              try {
                  Bitmap bitmap = MediaStore.Images.Media.getBitmap(getContentResolver(), imageUri);
                  ivProducto.setImageBitmap(bitmap);
              } catch (IOException e) {
                  e.printStackTrace();
                  Toast.makeText(this, "Error al cargar la imagen", Toast.LENGTH_SHORT).show();
              }
          }
      }
  
  
  
      private void cargarCategorias() {
          StringRequest stringRequest = new StringRequest(Request.Method.POST, URL,
                  response -> {
                      try {
                          JSONObject jsonResponse = new JSONObject(response);
                          JSONArray categorias = jsonResponse.getJSONArray("categorias");
                          listaCategorias.clear();
  
                          for (int i = 0; i < categorias.length(); i++) {
                              JSONObject categoria = categorias.getJSONObject(i);
                              listaCategorias.add(new CategoriaItem(
                                      categoria.getString("id"),
                                      categoria.getString("nombre")
                              ));
                          }
  
                          ArrayAdapter<CategoriaItem> adapter = new ArrayAdapter<>(
                                  this,
                                  android.R.layout.simple_dropdown_item_1line,
                                  listaCategorias);
                          spinnerCategoria.setAdapter(adapter);
                      } catch (Exception e) {
                          e.printStackTrace();
                          Toast.makeText(this, "Error al cargar categorías", Toast.LENGTH_SHORT).show();
                      }
                  },
                  error -> Toast.makeText(this, "Error de conexión", Toast.LENGTH_SHORT).show()) {
              @Override
              protected Map<String, String> getParams() {
                  Map<String, String> params = new HashMap<>();
                  params.put("operation", "getCategorias");
                  return params;
              }
          };
  
          Volley.newRequestQueue(this).add(stringRequest);
      }
  
      private void cargarProveedores() {
          StringRequest stringRequest = new StringRequest(Request.Method.POST, URL,
                  response -> {
                      try {
                          JSONObject jsonResponse = new JSONObject(response);
                          JSONArray proveedores = jsonResponse.getJSONArray("proveedores");
                          listaProveedores.clear();
  
                          for (int i = 0; i < proveedores.length(); i++) {
                              JSONObject proveedor = proveedores.getJSONObject(i);
                              listaProveedores.add(new ProveedorItem(
                                      proveedor.getString("id"),
                                      proveedor.getString("razon_social")
                              ));
                          }
  
                          ArrayAdapter<ProveedorItem> adapter = new ArrayAdapter<>(
                                  this,
                                  android.R.layout.simple_dropdown_item_1line,
                                  listaProveedores);
                          spinnerProveedor.setAdapter(adapter);
                      } catch (Exception e) {
                          e.printStackTrace();
                          Toast.makeText(this, "Error al cargar proveedores", Toast.LENGTH_SHORT).show();
                      }
                  },
                  error -> Toast.makeText(this, "Error de conexión", Toast.LENGTH_SHORT).show()) {
              @Override
              protected Map<String, String> getParams() {
                  Map<String, String> params = new HashMap<>();
                  params.put("operation", "getProveedores");
                  return params;
              }
          };
  
          Volley.newRequestQueue(this).add(stringRequest);
      }
  
      private void validarYGuardar() {
          String nombre = etNombre.getText().toString().trim();
          String descripcion = etDescripcion.getText().toString().trim();
          String precioStr = etPrecio.getText().toString().trim();
          String stockStr = etStock.getText().toString().trim();
  
          // Log de validación
          Log.d("VALIDACION", "Nombre: " + nombre);
          Log.d("VALIDACION", "Descripción: " + descripcion);
          Log.d("VALIDACION", "Precio: " + precioStr);
          Log.d("VALIDACION", "Stock: " + stockStr);
          Log.d("VALIDACION", "Categoria: " + (categoriaSeleccionada != null ?
                  categoriaSeleccionada.getId() : "null"));
          Log.d("VALIDACION", "Proveedor: " + (proveedorSeleccionado != null ?
                  proveedorSeleccionado.getId() : "null"));
  
          if (nombre.isEmpty()) {
              etNombre.setError("El nombre es requerido");
              return;
          }
  
          if (precioStr.isEmpty()) {
              etPrecio.setError("El precio es requerido");
              return;
          }
  
          if (stockStr.isEmpty()) {
              etStock.setError("El stock es requerido");
              return;
          }
  
          if (categoriaSeleccionada == null) {
              Toast.makeText(this, "Seleccione una categoría", Toast.LENGTH_SHORT).show();
              return;
          }
  
          if (proveedorSeleccionado == null) {
              Toast.makeText(this, "Seleccione un proveedor", Toast.LENGTH_SHORT).show();
              return;
          }
  
          // Proceder con el guardado
          guardarProducto(nombre, descripcion, precioStr, stockStr,
                  categoriaSeleccionada.getId(),
                  proveedorSeleccionado.getId());
      }
  
      // Modifica el método guardarProducto para incluir la subida de imagen
      private void guardarProducto(String nombre, String descripcion, String precio,
                                   String stock, String categoriaId, String proveedorId) {
          StringRequest stringRequest = new StringRequest(Request.Method.POST, URL,
                  response -> {
                      try {
                          Log.d("API_RESPONSE", response); // Agregar este log
                          JSONObject jsonResponse = new JSONObject(response);
                          boolean success = jsonResponse.getBoolean("success");
                          String message = jsonResponse.getString("message");
  
                          if (success) {
                              if (imageUri != null) {
                                  // Asegúrate de que el ID del producto está disponible
                                  String productoId = jsonResponse.optString("producto_id", "");
                                  if (!productoId.isEmpty()) {
                                      uploadImage(productoId);
                                  }
                              }
                              finish();
                          }
                          Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
                      } catch (Exception e) {
                          e.printStackTrace();
                          Log.e("API_ERROR", "Error parsing response: " + e.getMessage());
                          Toast.makeText(this, "Error: " + e.getMessage(), Toast.LENGTH_LONG).show();
                      }
                  },
                  error -> {
                      error.printStackTrace();
                      Log.e("API_ERROR", "Volley error: " + error.getMessage());
                      Toast.makeText(this, "Error de conexión: " + error.getMessage(),
                              Toast.LENGTH_LONG).show();
                  }) {
              @Override
              protected Map<String, String> getParams() {
                  Map<String, String> params = new HashMap<>();
                  params.put("operation", "addProducto");
                  params.put("nombre", nombre);
                  params.put("descripcion", descripcion);
                  params.put("precio", precio);
                  params.put("stock", stock);
                  params.put("categoria_id", categoriaId);
                  params.put("proveedor_id", proveedorId);
                  params.put("estado", switchEstado.isChecked() ? "1" : "0");
  
                  // Log de parámetros
                  Log.d("API_PARAMS", params.toString());
                  return params;
              }
          };
  
          Volley.newRequestQueue(this).add(stringRequest);
      }
  
      // Método para subir la imagen al servidor
      private void uploadImage(String productoId) {
          if (imageUri == null) return;
  
          // Crear un objeto File desde el Uri
          try {
              VolleyMultipartRequest multipartRequest = new VolleyMultipartRequest(Request.Method.POST, URL,
                      response -> {
                          String resultResponse = new String(response.data);
                          try {
                              JSONObject result = new JSONObject(resultResponse);
                              boolean success = result.getBoolean("success");
                              String message = result.getString("message");
  
                              if (!success) {
                                  runOnUiThread(() -> Toast.makeText(agregar_producto.this,
                                          "Error al subir la imagen: " + message,
                                          Toast.LENGTH_SHORT).show());
                              }
                          } catch (JSONException e) {
                              e.printStackTrace();
                          }
                      },
                      error -> Toast.makeText(agregar_producto.this,
                              "Error al subir la imagen",
                              Toast.LENGTH_SHORT).show()) {
                  @Override
                  protected Map<String, String> getParams() {
                      Map<String, String> params = new HashMap<>();
                      params.put("operation", "uploadProductImage");
                      params.put("id", productoId);
                      return params;
                  }
  
                  @Override
                  protected Map<String, DataPart> getByteData() {
                      Map<String, DataPart> params = new HashMap<>();
                      try {
                          String fileName = "product_image_" + System.currentTimeMillis() + ".jpg";
                          byte[] imageData = getBytes(agregar_producto.this, imageUri);
                          params.put("image", new DataPart(fileName, imageData, "image/jpeg"));
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                      return params;
                  }
              };
  
              Volley.newRequestQueue(this).add(multipartRequest);
          } catch (Exception e) {
              e.printStackTrace();
              Toast.makeText(this, "Error al procesar la imagen", Toast.LENGTH_SHORT).show();
          }
      }
  
      // Método auxiliar para convertir Uri a bytes
      public static byte[] getBytes(Context context, Uri uri) throws IOException {
          InputStream iStream = context.getContentResolver().openInputStream(uri);
          ByteArrayOutputStream byteBuffer = new ByteArrayOutputStream();
          int bufferSize = 1024;
          byte[] buffer = new byte[bufferSize];
  
          int len;
          while ((len = iStream.read(buffer)) != -1) {
              byteBuffer.write(buffer, 0, len);
          }
  
          return byteBuffer.toByteArray();
      }
  
  }

3. **JbStoreApp\app\src\main\java\jbstoreapp\com\editar_producto.java*

Linea::

  package jbstoreapp.com;
  
  import android.content.Intent;
  import android.net.Uri;
  import android.os.Bundle;
  import android.view.MenuItem;
  import android.widget.ArrayAdapter;
  import android.widget.AutoCompleteTextView;
  import android.widget.Button;
  import android.widget.EditText;
  import android.widget.ImageView;
  import android.widget.Toast;
  
  import androidx.activity.result.ActivityResultLauncher;
  import androidx.activity.result.PickVisualMediaRequest;
  import androidx.activity.result.contract.ActivityResultContracts;
  import androidx.appcompat.app.AppCompatActivity;
  import androidx.appcompat.widget.SwitchCompat;
  import androidx.appcompat.widget.Toolbar;
  
  import com.android.volley.Request;
  import com.android.volley.RequestQueue;
  import com.android.volley.toolbox.StringRequest;
  import com.android.volley.toolbox.Volley;
  import com.bumptech.glide.Glide;
  import com.github.dhaval2404.imagepicker.ImagePicker;
  
  import org.json.JSONArray;
  import org.json.JSONObject;
  
  import java.util.ArrayList;
  import java.util.HashMap;
  import java.util.Map;
  
  import android.content.Context;
  import android.net.Uri;
  import java.io.ByteArrayOutputStream;
  import java.io.IOException;
  import java.io.InputStream;
  import com.android.volley.NetworkResponse;
  import com.android.volley.Request;
  import com.android.volley.Response;
  import com.android.volley.VolleyError;
  import org.json.JSONException;
  import org.json.JSONObject;
  
  public class editar_producto extends AppCompatActivity {
      private EditText etNombre, etDescripcion, etPrecio, etStock;
      private AutoCompleteTextView spinnerCategoria, spinnerProveedor;
      private SwitchCompat switchEstado;
      private ImageView ivProducto;
      private Uri imageUri;
      private String productoId;
  
      private static final int PICK_IMAGE_REQUEST = 1;
      private ActivityResultLauncher<PickVisualMediaRequest> pickMedia;
  
  
      private String URL = "http:/192.168.125.59/api_inventario/api.php";
  
      private ArrayList<CategoriaItem> listaCategorias = new ArrayList<>();
      private ArrayList<ProveedorItem> listaProveedores = new ArrayList<>();
      private CategoriaItem categoriaSeleccionada;
      private ProveedorItem proveedorSeleccionado;
      private String currentCategoriaId;
      private String currentProveedorId;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_editar_producto);
  
          // Registrar el launcher para la selección de imágenes
          pickMedia = registerForActivityResult(new ActivityResultContracts.PickVisualMedia(), uri -> {
              if (uri != null) {
                  imageUri = uri;
                  ivProducto.setImageURI(uri);
              } else {
                  Toast.makeText(this, "No se seleccionó ninguna imagen", Toast.LENGTH_SHORT).show();
              }
          });
  
          // Configurar Toolbar
          Toolbar toolbar = findViewById(R.id.toolbar);
          setSupportActionBar(toolbar);
          getSupportActionBar().setDisplayHomeAsUpEnabled(true);
          getSupportActionBar().setTitle("Editar Producto");
  
          inicializarComponentes();
          cargarDatosProducto();
      }
  
      private void inicializarComponentes() {
          productoId = getIntent().getStringExtra("id");
  
          etNombre = findViewById(R.id.etNombre);
          etDescripcion = findViewById(R.id.etDescripcion);
          etPrecio = findViewById(R.id.etPrecio);
          etStock = findViewById(R.id.etStock);
          spinnerCategoria = findViewById(R.id.spinnerCategoria);
          spinnerProveedor = findViewById(R.id.spinnerProveedor);
          switchEstado = findViewById(R.id.switchEstado);
          ivProducto = findViewById(R.id.ivProducto);
  
          // Configurar Spinners
          spinnerCategoria.setOnItemClickListener((parent, view, position, id) -> {
              categoriaSeleccionada = listaCategorias.get(position);
          });
  
          spinnerProveedor.setOnItemClickListener((parent, view, position, id) -> {
              proveedorSeleccionado = listaProveedores.get(position);
          });
  
          Button btnGuardar = findViewById(R.id.btnGuardar);
          btnGuardar.setOnClickListener(v -> validarYActualizar());
      }
  
      private void cargarDatosProducto() {
          StringRequest stringRequest = new StringRequest(Request.Method.POST, URL,
                  response -> {
                      try {
                          JSONObject jsonResponse = new JSONObject(response);
                          JSONObject producto = jsonResponse.getJSONObject("producto");
  
                          etNombre.setText(producto.getString("nombre"));
                          etDescripcion.setText(producto.getString("descripcion"));
                          etPrecio.setText(String.valueOf(producto.getDouble("precio")));
                          etStock.setText(String.valueOf(producto.getInt("stock")));
  
                          currentCategoriaId = producto.getString("categoria_id");
                          currentProveedorId = producto.getString("proveedor_id");
  
                          switchEstado.setChecked(producto.getInt("estado") == 1);
  
                          // Cargar imagen si existe
                          String imagenUrl = producto.getString("imagen_url");
                          if (imagenUrl != null && !imagenUrl.isEmpty()) {
                              String imageUrl = "http://192.168.125.59/api_inventario/" + imagenUrl;
                              Glide.with(this)
                                      .load(imageUrl)
                                      .placeholder(R.drawable.logo)
                                      .error(imageUrl)
                                      .into(ivProducto);
                          } else {
                              ivProducto.setImageResource(R.drawable.logo);
                          }
  
                          // Cargar categorías y proveedores
                          cargarCategorias();
                          cargarProveedores();
  
                      } catch (Exception e) {
                          e.printStackTrace();
                          Toast.makeText(this, "Error al cargar datos", Toast.LENGTH_SHORT).show();
                      }
                  },
                  error -> Toast.makeText(this, "Error de conexión", Toast.LENGTH_SHORT).show()) {
              @Override
              protected Map<String, String> getParams() {
                  Map<String, String> params = new HashMap<>();
                  params.put("operation", "getProducto");
                  params.put("id", productoId);
                  return params;
              }
          };
  
          RequestQueue requestQueue = Volley.newRequestQueue(this);
          requestQueue.add(stringRequest);
      }
      private void cargarCategorias() {
          StringRequest stringRequest = new StringRequest(Request.Method.POST, URL,
                  response -> {
                      try {
                          JSONObject jsonResponse = new JSONObject(response);
                          JSONArray categorias = jsonResponse.getJSONArray("categorias");
                          listaCategorias.clear();
  
                          int selectedPosition = -1;
                          for (int i = 0; i < categorias.length(); i++) {
                              JSONObject categoria = categorias.getJSONObject(i);
                              CategoriaItem item = new CategoriaItem(
                                      categoria.getString("id"),
                                      categoria.getString("nombre")
                              );
                              listaCategorias.add(item);
  
                              if (item.getId().equals(currentCategoriaId)) {
                                  selectedPosition = i;
                                  categoriaSeleccionada = item;
                              }
                          }
  
                          ArrayAdapter<CategoriaItem> adapter = new ArrayAdapter<>(
                                  this,
                                  android.R.layout.simple_dropdown_item_1line,
                                  listaCategorias);
                          spinnerCategoria.setAdapter(adapter);
  
                          if (selectedPosition != -1) {
                              spinnerCategoria.setText(listaCategorias.get(selectedPosition).toString(), false);
                          }
                      } catch (Exception e) {
                          e.printStackTrace();
                          Toast.makeText(this, "Error al cargar categorías", Toast.LENGTH_SHORT).show();
                      }
                  },
                  error -> Toast.makeText(this, "Error de conexión", Toast.LENGTH_SHORT).show()) {
              @Override
              protected Map<String, String> getParams() {
                  Map<String, String> params = new HashMap<>();
                  params.put("operation", "getCategorias");
                  return params;
              }
          };
  
          Volley.newRequestQueue(this).add(stringRequest);
      }
  
      private void cargarProveedores() {
          StringRequest stringRequest = new StringRequest(Request.Method.POST, URL,
                  response -> {
                      try {
                          JSONObject jsonResponse = new JSONObject(response);
                          JSONArray proveedores = jsonResponse.getJSONArray("proveedores");
                          listaProveedores.clear();
  
                          int selectedPosition = -1;
                          for (int i = 0; i < proveedores.length(); i++) {
                              JSONObject proveedor = proveedores.getJSONObject(i);
                              ProveedorItem item = new ProveedorItem(
                                      proveedor.getString("id"),
                                      proveedor.getString("razon_social")
                              );
                              listaProveedores.add(item);
  
                              if (item.getId().equals(currentProveedorId)) {
                                  selectedPosition = i;
                                  proveedorSeleccionado = item;
                              }
                          }
  
                          ArrayAdapter<ProveedorItem> adapter = new ArrayAdapter<>(
                                  this,
                                  android.R.layout.simple_dropdown_item_1line,
                                  listaProveedores);
                          spinnerProveedor.setAdapter(adapter);
  
                          if (selectedPosition != -1) {
                              spinnerProveedor.setText(listaProveedores.get(selectedPosition).toString(), false);
                          }
                      } catch (Exception e) {
                          e.printStackTrace();
                          Toast.makeText(this, "Error al cargar proveedores", Toast.LENGTH_SHORT).show();
                      }
                  },
                  error -> Toast.makeText(this, "Error de conexión", Toast.LENGTH_SHORT).show()) {
              @Override
              protected Map<String, String> getParams() {
                  Map<String, String> params = new HashMap<>();
                  params.put("operation", "getProveedores");
                  return params;
              }
          };
  
          Volley.newRequestQueue(this).add(stringRequest);
      }
  
      private void validarYActualizar() {
          String nombre = etNombre.getText().toString().trim();
          String descripcion = etDescripcion.getText().toString().trim();
          String precioStr = etPrecio.getText().toString().trim();
          String stockStr = etStock.getText().toString().trim();
  
          if (nombre.isEmpty()) {
              etNombre.setError("El nombre es requerido");
              return;
          }
  
          if (precioStr.isEmpty()) {
              etPrecio.setError("El precio es requerido");
              return;
          }
  
          if (stockStr.isEmpty()) {
              etStock.setError("El stock es requerido");
              return;
          }
  
          String categoriaId = categoriaSeleccionada != null ?
                  categoriaSeleccionada.getId() : currentCategoriaId;
          String proveedorId = proveedorSeleccionado != null ?
                  proveedorSeleccionado.getId() : currentProveedorId;
  
          actualizarProducto(nombre, descripcion, precioStr, stockStr,
                  categoriaId, proveedorId);
      }
  
      private void actualizarProducto(String nombre, String descripcion, String precio,
                                      String stock, String categoriaId, String proveedorId) {
          StringRequest stringRequest = new StringRequest(Request.Method.POST, URL,
                  response -> {
                      try {
                          JSONObject jsonResponse = new JSONObject(response);
                          boolean success = jsonResponse.getBoolean("success");
                          String message = jsonResponse.getString("message");
  
                          Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
                          if (success) {
                              finish();
                          }
  
                          if (success) {
                              // Si hay una nueva imagen seleccionada, súbela
                              if (imageUri != null) {
                                  uploadImage(productoId);
                              }
                              finish();
                          }
  
                      } catch (Exception e) {
                          e.printStackTrace();
                          Toast.makeText(this, "Error en el servidor", Toast.LENGTH_SHORT).show();
                      }
                  },
                  error -> Toast.makeText(this, "Error de conexión", Toast.LENGTH_SHORT).show()) {
              @Override
              protected Map<String, String> getParams() {
                  Map<String, String> params = new HashMap<>();
                  params.put("operation", "updateProducto");
                  params.put("id", productoId);
                  params.put("nombre", nombre);
                  params.put("descripcion", descripcion);
                  params.put("precio", precio);
                  params.put("stock", stock);
                  params.put("categoria_id", categoriaId);
                  params.put("proveedor_id", proveedorId);
                  params.put("estado", switchEstado.isChecked() ? "1" : "0");
                  return params;
              }
          };
  
          Volley.newRequestQueue(this).add(stringRequest);
      }
  
      @Override
      public boolean onOptionsItemSelected(MenuItem item) {
          if (item.getItemId() == android.R.id.home) {
              onBackPressed();
              return true;
          }
          return super.onOptionsItemSelected(item);
      }
  
      // Método para subir la imagen al servidor
      private void uploadImage(String productoId) {
          if (imageUri == null) return;
  
          // Crear un objeto File desde el Uri
          try {
              VolleyMultipartRequest multipartRequest = new VolleyMultipartRequest(Request.Method.POST, URL,
                      response -> {
                          String resultResponse = new String(response.data);
                          try {
                              JSONObject result = new JSONObject(resultResponse);
                              boolean success = result.getBoolean("success");
                              String message = result.getString("message");
  
                              if (!success) {
                                  runOnUiThread(() -> Toast.makeText(editar_producto.this,
                                          "Error al subir la imagen: " + message,
                                          Toast.LENGTH_SHORT).show());
                              }
                          } catch (JSONException e) {
                              e.printStackTrace();
                          }
                      },
                      error -> Toast.makeText(editar_producto.this,
                              "Error al subir la imagen",
                              Toast.LENGTH_SHORT).show()) {
                  @Override
                  protected Map<String, String> getParams() {
                      Map<String, String> params = new HashMap<>();
                      params.put("operation", "uploadProductImage");
                      params.put("id", productoId);
                      return params;
                  }
  
                  @Override
                  protected Map<String, DataPart> getByteData() {
                      Map<String, DataPart> params = new HashMap<>();
                      try {
                          String fileName = "product_image_" + System.currentTimeMillis() + ".jpg";
                          byte[] imageData = getBytes(editar_producto.this, imageUri);
                          params.put("image", new DataPart(fileName, imageData, "image/jpeg"));
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                      return params;
                  }
              };
  
              Volley.newRequestQueue(this).add(multipartRequest);
          } catch (Exception e) {
              e.printStackTrace();
              Toast.makeText(this, "Error al procesar la imagen", Toast.LENGTH_SHORT).show();
          }
      }
  
      // Método auxiliar para convertir Uri a bytes
      public static byte[] getBytes(Context context, Uri uri) throws IOException {
          InputStream iStream = context.getContentResolver().openInputStream(uri);
          ByteArrayOutputStream byteBuffer = new ByteArrayOutputStream();
          int bufferSize = 1024;
          byte[] buffer = new byte[bufferSize];
  
          int len;
          while ((len = iStream.read(buffer)) != -1) {
              byteBuffer.write(buffer, 0, len);
          }
  
          return byteBuffer.toByteArray();
      }
  }

4.  **api.php**

Linea::

  <?php
  header('Content-Type: application/json');
  require_once 'config.php';
  
  // Obtener el tipo de operación
  $operation = isset($_POST['operation']) ? cleanInput($_POST['operation']) : '';
  
  function uploadImage($file, $targetDir = "uploads/") {
      if (!file_exists($targetDir)) {
          mkdir($targetDir, 0777, true);
      }
      
      $fileName = uniqid() . '_' . basename($file["name"]);
      $targetPath = $targetDir . $fileName;
      
      if (move_uploaded_file($file["tmp_name"], $targetPath)) {
          return $targetPath;
      }
      return false;
  }
  
  switch($operation) {
      // OPERACIONES PARA CATEGORÍAS
      case 'getCategorias':
          $sql = "SELECT * FROM categorias ORDER BY nombre ASC";
          $result = $conn->query($sql);
          $categorias = array();
  
          if ($result->num_rows > 0) {
              while($row = $result->fetch_assoc()) {
                  $categorias[] = $row;
              }
          }
  
          echo json_encode([
              'success' => true,
              'categorias' => $categorias
          ]);
          break;
  
      case 'addCategoria':
          $nombre = isset($_POST['nombre']) ? cleanInput($_POST['nombre']) : '';
          $descripcion = isset($_POST['descripcion']) ? cleanInput($_POST['descripcion']) : '';
  
          if (empty($nombre)) {
              echo json_encode([
                  'success' => false,
                  'message' => 'El nombre es requerido'
              ]);
              break;
          }
  
          $stmt = $conn->prepare("INSERT INTO categorias (nombre, descripcion) VALUES (?, ?)");
          $stmt->bind_param("ss", $nombre, $descripcion);
  
          if ($stmt->execute()) {
              echo json_encode([
                  'success' => true,
                  'message' => 'Categoría agregada correctamente'
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al agregar la categoría'
              ]);
          }
          break;
  
      case 'updateCategoria':
          $id = isset($_POST['id']) ? cleanInput($_POST['id']) : '';
          $nombre = isset($_POST['nombre']) ? cleanInput($_POST['nombre']) : '';
          $descripcion = isset($_POST['descripcion']) ? cleanInput($_POST['descripcion']) : '';
  
          if (empty($id) || empty($nombre)) {
              echo json_encode([
                  'success' => false,
                  'message' => 'ID y nombre son requeridos'
              ]);
              break;
          }
  
          $stmt = $conn->prepare("UPDATE categorias SET nombre = ?, descripcion = ? WHERE id = ?");
          $stmt->bind_param("ssi", $nombre, $descripcion, $id);
  
          if ($stmt->execute()) {
              echo json_encode([
                  'success' => true,
                  'message' => 'Categoría actualizada correctamente'
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al actualizar la categoría'
              ]);
          }
          break;
  
      case 'deleteCategoria':
          $id = isset($_POST['id']) ? cleanInput($_POST['id']) : '';
  
          if (empty($id)) {
              echo json_encode([
                  'success' => false,
                  'message' => 'ID es requerido'
              ]);
              break;
          }
  
          // Primero verificar si hay productos asociados
          $stmt = $conn->prepare("SELECT COUNT(*) as count FROM productos WHERE categoria_id = ?");
          $stmt->bind_param("i", $id);
          $stmt->execute();
          $result = $stmt->get_result();
          $row = $result->fetch_assoc();
  
          if ($row['count'] > 0) {
              echo json_encode([
                  'success' => false,
                  'message' => 'No se puede eliminar la categoría porque tiene productos asociados'
              ]);
              break;
          }
  
          $stmt = $conn->prepare("DELETE FROM categorias WHERE id = ?");
          $stmt->bind_param("i", $id);
  
          if ($stmt->execute()) {
              echo json_encode([
                  'success' => true,
                  'message' => 'Categoría eliminada correctamente'
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al eliminar la categoría'
              ]);
          }
          break;
  
      // OPERACIONES PARA PROVEEDORES
      case 'getProveedores':
          $sql = "SELECT * FROM proveedores ORDER BY razon_social ASC";
          $result = $conn->query($sql);
          $proveedores = array();
  
          if ($result->num_rows > 0) {
              while($row = $result->fetch_assoc()) {
                  $proveedores[] = $row;
              }
          }
  
          echo json_encode([
              'success' => true,
              'proveedores' => $proveedores
          ]);
          break;
  
      case 'addProveedor':
          $ruc = isset($_POST['ruc']) ? cleanInput($_POST['ruc']) : '';
          $razon_social = isset($_POST['razon_social']) ? cleanInput($_POST['razon_social']) : '';
          $direccion = isset($_POST['direccion']) ? cleanInput($_POST['direccion']) : '';
          $telefono = isset($_POST['telefono']) ? cleanInput($_POST['telefono']) : '';
          $email = isset($_POST['email']) ? cleanInput($_POST['email']) : '';
          $persona_contacto = isset($_POST['persona_contacto']) ? cleanInput($_POST['persona_contacto']) : '';
          $estado = isset($_POST['estado']) ? cleanInput($_POST['estado']) : '1';
  
          // Validar RUC único
          $stmt = $conn->prepare("SELECT id FROM proveedores WHERE ruc = ?");
          $stmt->bind_param("s", $ruc);
          $stmt->execute();
          $result = $stmt->get_result();
  
          if ($result->num_rows > 0) {
              echo json_encode([
                  'success' => false,
                  'message' => 'El RUC ya está registrado'
              ]);
              break;
          }
  
          $stmt = $conn->prepare("INSERT INTO proveedores (ruc, razon_social, direccion, telefono, email, persona_contacto, estado) VALUES (?, ?, ?, ?, ?, ?, ?)");
          $stmt->bind_param("ssssssi", $ruc, $razon_social, $direccion, $telefono, $email, $persona_contacto, $estado);
  
          if ($stmt->execute()) {
              echo json_encode([
                  'success' => true,
                  'message' => 'Proveedor agregado correctamente'
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al agregar el proveedor'
              ]);
          }
          break;
  
      case 'updateProveedor':
          $id = isset($_POST['id']) ? cleanInput($_POST['id']) : '';
          $ruc = isset($_POST['ruc']) ? cleanInput($_POST['ruc']) : '';
          $razon_social = isset($_POST['razon_social']) ? cleanInput($_POST['razon_social']) : '';
          $direccion = isset($_POST['direccion']) ? cleanInput($_POST['direccion']) : '';
          $telefono = isset($_POST['telefono']) ? cleanInput($_POST['telefono']) : '';
          $email = isset($_POST['email']) ? cleanInput($_POST['email']) : '';
          $persona_contacto = isset($_POST['persona_contacto']) ? cleanInput($_POST['persona_contacto']) : '';
          $estado = isset($_POST['estado']) ? cleanInput($_POST['estado']) : '1';
  
          // Validar RUC único excluyendo el proveedor actual
          $stmt = $conn->prepare("SELECT id FROM proveedores WHERE ruc = ? AND id != ?");
          $stmt->bind_param("si", $ruc, $id);
          $stmt->execute();
          $result = $stmt->get_result();
  
          if ($result->num_rows > 0) {
              echo json_encode([
                  'success' => false,
                  'message' => 'El RUC ya está registrado por otro proveedor'
              ]);
              break;
          }
  
          $stmt = $conn->prepare("UPDATE proveedores SET ruc = ?, razon_social = ?, direccion = ?, telefono = ?, email = ?, persona_contacto = ?, estado = ? WHERE id = ?");
          $stmt->bind_param("ssssssii", $ruc, $razon_social, $direccion, $telefono, $email, $persona_contacto, $estado, $id);
  
          if ($stmt->execute()) {
              echo json_encode([
                  'success' => true,
                  'message' => 'Proveedor actualizado correctamente'
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al actualizar el proveedor'
              ]);
          }
          break;
  
      case 'deleteProveedor':
          $id = isset($_POST['id']) ? cleanInput($_POST['id']) : '';
  
          if (empty($id)) {
              echo json_encode([
                  'success' => false,
                  'message' => 'ID es requerido'
              ]);
              break;
          }
  
          // Verificar si hay productos asociados
          $stmt = $conn->prepare("SELECT COUNT(*) as count FROM productos WHERE proveedor_id = ?");
          $stmt->bind_param("i", $id);
          $stmt->execute();
          $result = $stmt->get_result();
          $row = $result->fetch_assoc();
  
          if ($row['count'] > 0) {
              echo json_encode([
                  'success' => false,
                  'message' => 'No se puede eliminar el proveedor porque tiene productos asociados'
              ]);
              break;
          }
  
          $stmt = $conn->prepare("DELETE FROM proveedores WHERE id = ?");
          $stmt->bind_param("i", $id);
  
          if ($stmt->execute()) {
              echo json_encode([
                  'success' => true,
                  'message' => 'Proveedor eliminado correctamente'
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al eliminar el proveedor'
              ]);
          }
          break;
  
      default:
          echo json_encode([
              'success' => false,
              'message' => 'Operación no válida'
          ]);
          break;
  
  
  case 'getProductos':
      $sql = "SELECT p.*, c.nombre as categoria_nombre, pr.razon_social as proveedor_nombre 
              FROM productos p 
              LEFT JOIN categorias c ON p.categoria_id = c.id 
              LEFT JOIN proveedores pr ON p.proveedor_id = pr.id 
              ORDER BY p.nombre ASC";
      $result = $conn->query($sql);
      $productos = array();
      
      if ($result->num_rows > 0) {
          while($row = $result->fetch_assoc()) {
              // Asegúrate de que la URL de la imagen sea accesible
              if (!empty($row['imagen_url'])) {
                  $row['imagen_url'] = $row['imagen_url']; // La ruta relativa desde la raíz del servidor
              }
              $productos[] = $row;
          }
      }
      
      echo json_encode([
          'success' => true,
          'productos' => $productos
      ]);
      break;
  
  case 'addProducto':
      $nombre = isset($_POST['nombre']) ? cleanInput($_POST['nombre']) : '';
      $descripcion = isset($_POST['descripcion']) ? cleanInput($_POST['descripcion']) : '';
      $precio = isset($_POST['precio']) ? cleanInput($_POST['precio']) : '0';
      $stock = isset($_POST['stock']) ? cleanInput($_POST['stock']) : '0';
      $categoria_id = isset($_POST['categoria_id']) ? cleanInput($_POST['categoria_id']) : null;
      $proveedor_id = isset($_POST['proveedor_id']) ? cleanInput($_POST['proveedor_id']) : null;
      $estado = isset($_POST['estado']) ? cleanInput($_POST['estado']) : '1';
      
      // Log de datos recibidos
      error_log("Datos recibidos en addProducto: " . print_r($_POST, true));
      
      if (empty($nombre)) {
          echo json_encode([
              'success' => false,
              'message' => 'El nombre es requerido'
          ]);
          break;
      }
      
      try {
          $stmt = $conn->prepare("INSERT INTO productos (nombre, descripcion, precio, stock, categoria_id, proveedor_id, estado) VALUES (?, ?, ?, ?, ?, ?, ?)");
          $stmt->bind_param("ssdiiii", $nombre, $descripcion, $precio, $stock, $categoria_id, $proveedor_id, $estado);
          
          if ($stmt->execute()) {
              $producto_id = $conn->insert_id;
              echo json_encode([
                  'success' => true,
                  'message' => 'Producto agregado correctamente',
                  'producto_id' => $producto_id
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al agregar el producto: ' . $stmt->error
              ]);
          }
      } catch (Exception $e) {
          error_log("Error en addProducto: " . $e->getMessage());
          echo json_encode([
              'success' => false,
              'message' => 'Error en el servidor: ' . $e->getMessage()
          ]);
      }
      break;
  
  case 'deleteProducto':
          $id = isset($_POST['id']) ? cleanInput($_POST['id']) : '';
          
          if (empty($id)) {
              echo json_encode([
                  'success' => false,
                  'message' => 'ID es requerido'
              ]);
              break;
          }
          
          $stmt = $conn->prepare("DELETE FROM productos WHERE id = ?");
          $stmt->bind_param("i", $id);
          
          if ($stmt->execute()) {
              echo json_encode([
                  'success' => true,
                  'message' => 'Producto eliminado correctamente'
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al eliminar el producto'
              ]);
          }
          break;
  
      case 'getProducto':
          $id = isset($_POST['id']) ? cleanInput($_POST['id']) : '';
          
          if (empty($id)) {
              echo json_encode([
                  'success' => false,
                  'message' => 'ID es requerido'
              ]);
              break;
          }
          
          $stmt = $conn->prepare("SELECT p.*, c.nombre as categoria_nombre, 
                                 pr.razon_social as proveedor_nombre 
                                 FROM productos p 
                                 LEFT JOIN categorias c ON p.categoria_id = c.id 
                                 LEFT JOIN proveedores pr ON p.proveedor_id = pr.id 
                                 WHERE p.id = ?");
          $stmt->bind_param("i", $id);
          $stmt->execute();
          $result = $stmt->get_result();
          
          if ($row = $result->fetch_assoc()) {
              echo json_encode([
                  'success' => true,
                  'producto' => $row
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Producto no encontrado'
              ]);
          }
          break;
  
      case 'updateProducto':
          $id = isset($_POST['id']) ? cleanInput($_POST['id']) : '';
          $nombre = isset($_POST['nombre']) ? cleanInput($_POST['nombre']) : '';
          $descripcion = isset($_POST['descripcion']) ? cleanInput($_POST['descripcion']) : '';
          $precio = isset($_POST['precio']) ? cleanInput($_POST['precio']) : '0';
          $stock = isset($_POST['stock']) ? cleanInput($_POST['stock']) : '0';
          $categoria_id = isset($_POST['categoria_id']) ? cleanInput($_POST['categoria_id']) : null;
          $proveedor_id = isset($_POST['proveedor_id']) ? cleanInput($_POST['proveedor_id']) : null;
          $estado = isset($_POST['estado']) ? cleanInput($_POST['estado']) : '1';
          
          if (empty($id) || empty($nombre)) {
              echo json_encode([
                  'success' => false,
                  'message' => 'ID y nombre son requeridos'
              ]);
              break;
          }
          
          $stmt = $conn->prepare("UPDATE productos SET nombre = ?, descripcion = ?, 
                                 precio = ?, stock = ?, categoria_id = ?, proveedor_id = ?, 
                                 estado = ? WHERE id = ?");
          $stmt->bind_param("ssdiiiis", $nombre, $descripcion, $precio, $stock, 
                            $categoria_id, $proveedor_id, $estado, $id);
          
          if ($stmt->execute()) {
              echo json_encode([
                  'success' => true,
                  'message' => 'Producto actualizado correctamente'
              ]);
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al actualizar el producto'
              ]);
          }
          break;
  
    case 'uploadProductImage':
      if (isset($_FILES['image'])) {
          $uploadDir = "uploads/"; // Directorio relativo a la ubicación del API
          $uploadResult = uploadImage($_FILES['image'], $uploadDir);
          if ($uploadResult) {
              $id = isset($_POST['id']) ? cleanInput($_POST['id']) : '';
              
              if (!empty($id)) {
                  // Guardar la ruta relativa
                  $stmt = $conn->prepare("UPDATE productos SET imagen_url = ? WHERE id = ?");
                  $stmt->bind_param("si", $uploadResult, $id);
                  
                  if ($stmt->execute()) {
                      echo json_encode([
                          'success' => true,
                          'message' => 'Imagen actualizada correctamente',
                          'imagen_url' => $uploadResult
                      ]);
                  } else {
                      echo json_encode([
                          'success' => false,
                          'message' => 'Error al actualizar la imagen'
                      ]);
                  }
              }
          } else {
              echo json_encode([
                  'success' => false,
                  'message' => 'Error al subir la imagen'
              ]);
          }
      } else {
          echo json_encode([
              'success' => false,
              'message' => 'No se recibió ninguna imagen'
          ]);
      }
      break;
  }
  
  $conn->close();
  ?>

**TENER EN CUENTA LO QUE SE HA RELIZADO**

Desde el problema de selección de imágenes, se modificaron los siguientes archivos:

1. AndroidManifest.xml
- Se actualizaron los permisos para Android 13+ (READ_MEDIA_IMAGES)
- Se eliminaron permisos obsoletos de storage

2. agregar_producto.java y editar_producto.java
- Se implementó ActivityResultLauncher para la selección de imágenes
- Se agregó la lógica de gestión de permisos
- Se implementó el método uploadImage para subir imágenes al servidor
- Se añadió manejo de errores y logs

3. api.php
- Se añadió manejo de subida de imágenes
- Se creó la función uploadImage
- Se modificó el procesamiento de respuestas incluyendo URLs de imágenes
- Se agregó manejo de errores detallado

4. Base de datos
- Se verificó la estructura para almacenar URLs de imágenes

5. Servidor
- Se creó directorio 'uploads' para almacenar imágenes
- Se configuraron permisos necesarios

