# Documentación del Proyecto — marketplace_main  
## Actualizaciones del Parcial 2

Este documento describe de forma detallada todas las actualizaciones realizadas al proyecto **marketplace_main** durante el parcial 2. Incluye explicaciones, código y funcionamiento dentro de la aplicación *store*.

---

# 1. Actualización del archivo `forms.py`

## 🔹 LoginForm
Permite que un usuario existente pueda iniciar sesión ingresando su nombre de usuario y contraseña.

### **Código:**
```python
# Importamos Django Forms y el modelo User
from django import forms
from django.contrib.auth.models import User
from .models import Item

# Formulario para iniciar sesión
class LoginForm(forms.Form):
    username = forms.CharField(max_length=150)  # Campo para el nombre de usuario
    password = forms.CharField(widget=forms.PasswordInput)  # Campo tipo contraseña
🔹 SignupForm
Formulario utilizado para registrar nuevos usuarios.

Código:
python
Copiar código
# Formulario de registro de nuevos usuarios
class SignupForm(forms.ModelForm):
    password = forms.CharField(widget=forms.PasswordInput)  # La contraseña se oculta al escribir

    class Meta:
        model = User  # Modelo de Django para usuarios
        fields = ['username', 'email', 'password']  # Campos que aparecerán en el formulario
🔹 NewItemForm
Formulario para agregar nuevos artículos al marketplace.

Código:
python
Copiar código
# Formulario basado en el modelo Item
class NewItemForm(forms.ModelForm):
    class Meta:
        model = Item  # Modelo que representa los artículos
        fields = ['name', 'description', 'price', 'image']  # Campos incluidos en el formulario
2. Actualizaciones del archivo views.py
🔹 login()
Vista encargada de autenticar usuarios.

Código:
python
Copiar código
# Importaciones necesarias
from django.shortcuts import render, redirect
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required
from .forms import LoginForm, SignupForm, NewItemForm
from .models import Item

# Vista para iniciar sesión
def login(request):
    form = LoginForm(request.POST or None)  # Recibimos los datos del formulario
    if request.method == 'POST' and form.is_valid():  # Si enviaron formulario y es válido
        user = authenticate(
            username=form.cleaned_data['username'],  # Verificamos usuario
            password=form.cleaned_data['password']   # Verificamos contraseña
        )
        if user:  # Si las credenciales son correctas
            login(request, user)  # Inicia sesión
            return redirect('store:home')  # Redirige al inicio
    return render(request, 'store/login.html', {'form': form})
🔹 logout_user()
Cierra la sesión del usuario actual.

Código:
python
Copiar código
def logout_user(request):
    logout(request)  # Django cierra la sesión del usuario
    return redirect('store:home')  # Redirige a la página principal
🔹 detail()
Muestra la información detallada de un artículo.

Código:
python
Copiar código
def detail(request, pk):
    item = Item.objects.get(id=pk)  # Obtiene el artículo por ID
    return render(request, 'store/item.html', {'item': item})  # Renderiza la plantilla
🔹 add_item()
Vista para agregar artículos, disponible solo para usuarios autenticados.

Código:
python
Copiar código
@login_required  # Decorador para restringir acceso
def add_item(request):
    form = NewItemForm(request.POST or None, request.FILES or None)  # Recibe datos y archivos (imagen)
    if request.method == 'POST' and form.is_valid():  # Si el formulario es válido
        form.save()  # Guarda el nuevo artículo
        return redirect('store:home')  # Redirige al inicio
    return render(request, 'store/form.html', {'form': form})
3. Explicación del decorador @login_required
Este decorador evita que usuarios NO autenticados accedan a vistas protegidas.

Cuando se intenta entrar a una vista protegida sin iniciar sesión, Django redirige automáticamente a /login/.

Ejemplo usado en el proyecto:
python
Copiar código
@login_required
def add_item(request):
    ...
4. Actualizaciones del archivo urls.py
Se agregaron rutas para las nuevas funcionalidades.

Código:
python
Copiar código
from django.urls import path
from . import views

app_name = 'store'  # Espacio de nombres de la app

urlpatterns = [
    path('login/', views.login, name='login'),  # Login
    path('logout/', views.logout_user, name='logout'),  # Logout
    path('item/<int:pk>/', views.detail, name='detail'),  # Detalle de artículo
    path('add/', views.add_item, name='add_item'),  # Agregar artículo
]
5. Actualizaciones en store/templates
🔹 item.html
html
Copiar código
<h1>{{ item.name }}</h1>
<p>{{ item.description }}</p>
<p>Precio: ${{ item.price }}</p>
<img src="{{ item.image.url }}" width="200">
🔹 login.html
html
Copiar código
<h2>Iniciar Sesión</h2>
<form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Entrar</button>
</form>
🔹 signup.html
html
Copiar código
<h2>Crear Cuenta</h2>
<form method="POST">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Registrarse</button>
</form>
🔹 navigation.html
html
Copiar código
<nav>
    <a href="/">Inicio</a>

    {% if user.is_authenticated %}
        <a href="/add/">Agregar Artículo</a>
        <a href="/logout/">Cerrar Sesión</a>
    {% else %}
        <a href="/login/">Iniciar Sesión</a>
        <a href="/signup/">Registrarse</a>
    {% endif %}
</nav>
🔹 form.html
html
Copiar código
<h2>Nuevo Artículo</h2>
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Guardar</button>
</form>
