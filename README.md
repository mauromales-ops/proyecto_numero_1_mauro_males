# -*- coding: utf-8 -*-
"""
Archivo: proyecto_numero_1_mauro_males.py
Descripción:
Crea una aplicación web completa con Flask, genera los archivos necesarios
(templates, CSS y contactos.txt) y arranca la app en http://localhost:8000

Requisitos:
- Python 3.10+
- Flask 2.2.5 → pip install Flask==2.2.5

Uso:
1) python proyecto_numero_1_mauro_males.py
2) Luego ejecuta: python app.py
"""

from pathlib import Path
import textwrap
import os

BASE = Path.cwd()

# ======================== CONTENIDO DE app.py ========================
APP_PY = textwrap.dedent('''
from flask import Flask, render_template, url_for, request, Response, redirect
from datetime import datetime
import os

app = Flask(__name__, static_folder='static', template_folder='templates')

SERVICES = [
    {"title": "Obra Civil", "desc": "Construcción de viviendas, edificios y naves industriales."},
    {"title": "Remodelaciones", "desc": "Refacciones, ampliaciones y modernización de espacios."},
    {"title": "Gestión de Proyectos", "desc": "Planificación, cronograma y control de calidad."},
]

PROJECTS = [
    {"name": "Edificio Central", "summary": "Edificio de oficinas - 8 pisos"},
    {"name": "Parque Residencial", "summary": "Conjunto de 24 viviendas unifamiliares"},
]

@app.route("/")
def index():
    meta = {"title": "Constructora Ejemplo - Inicio", "description": "Servicios de construcción y reformas."}
    return render_template("index.html", services=SERVICES, projects=PROJECTS, meta=meta)

@app.route("/servicios")
def servicios():
    meta = {"title": "Servicios - Constructora Ejemplo"}
    return render_template("servicios.html", services=SERVICES, meta=meta)

@app.route("/proyectos")
def proyectos():
    meta = {"title": "Proyectos - Constructora Ejemplo"}
    return render_template("proyectos.html", projects=PROJECTS, meta=meta)

@app.route("/contacto", methods=["GET", "POST"])
def contacto():
    meta = {"title": "Contacto - Constructora Ejemplo"}
    if request.method == "POST":
        name = request.form.get("name")
        email = request.form.get("email")
        body = request.form.get("message")
        timestamp = datetime.utcnow().isoformat()
        line = f"{timestamp} | {name} | {email} | {body}\\n"
        with open('contactos.txt', 'a', encoding='utf-8') as f:
            f.write(line)
        return redirect(url_for('contacto') + '#gracias')
    return render_template("contacto.html", meta=meta)

@app.route('/robots.txt')
def robots_txt():
    host = request.url_root.rstrip('/')
    txt = f"User-agent: *\\nAllow: /\\nSitemap: {host}/sitemap.xml\\n"
    return Response(txt, mimetype='text/plain')

@app.route('/sitemap.xml')
def sitemap():
    host = request.url_root.rstrip('/')
    pages = [
        (host + url_for('index'), datetime.utcnow().date().isoformat()),
        (host + url_for('servicios'), datetime.utcnow().date().isoformat()),
        (host + url_for('proyectos'), datetime.utcnow().date().isoformat()),
        (host + url_for('contacto'), datetime.utcnow().date().isoformat()),
    ]
    xml_items = []
    for loc, lastmod in pages:
        xml_items.append(
            f"<url><loc>{loc}</loc><lastmod>{lastmod}</lastmod><changefreq>weekly</changefreq><priority>0.7</priority></url>"
        )
    sitemap_xml = "<?xml version='1.0' encoding='UTF-8'?>\\n" \\
                  "<urlset xmlns='http://www.sitemaps.org/schemas/sitemap/0.9'>\\n" \\
                  + "\\n".join(xml_items) + "\\n</urlset>"
    return Response(sitemap_xml, mimetype='application/xml')

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 8000))
    app.run(host='0.0.0.0', port=port, debug=True)
''')

# ======================== PLANTILLAS HTML ========================
TEMPLATES = {
    'base.html': textwrap.dedent('''
<!doctype html>
<html lang="es">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>{% block title %}{{ meta.title if meta and meta.title else 'Constructora Ejemplo' }}{% endblock %}</title>
<meta name="description" content="{{ meta.description if meta and meta.description else '' }}">
<link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
<header class="site-header">
<div class="container header-inner">
<h1 class="brand"><a href="{{ url_for('index') }}">Constructora Ejemplo</a></h1>
<nav class="main-nav">
<a href="{{ url_for('index') }}">Inicio</a>
<a href="{{ url_for('servicios') }}">Servicios</a>
<a href="{{ url_for('proyectos') }}">Proyectos</a>
<a href="{{ url_for('contacto') }}">Contacto</a>
</nav>
</div>
</header>
<main class="container">{% block content %}{% endblock %}</main>
<footer class="site-footer"><div class="container"><p>© 2025 Constructora Ejemplo</p></div></footer>
</body>
</html>
'''),

    'index.html': textwrap.dedent('''
{% extends 'base.html' %}
{% block title %}Inicio - Constructora Ejemplo{% endblock %}
{% block content %}
<section class="hero">
<h2>Construimos tus ideas</h2>
<p>Obras civiles, remodelaciones y gestión integral de proyectos.</p>
<a class="btn" href="{{ url_for('servicios') }}">Nuestros Servicios</a>
</section>
<section class="cards">
<h3>Servicios destacados</h3>
<div class="grid">
{% for s in services %}
<article class="card"><h4>{{ s.title }}</h4><p>{{ s.desc }}</p></article>
{% endfor %}
</div>
</section>
<section class="projects">
<h3>Proyectos recientes</h3>
<ul>{% for p in projects %}<li><strong>{{ p.name }}</strong> — {{ p.summary }}</li>{% endfor %}</ul>
</section>
{% endblock %}
'''),

    'servicios.html': textwrap.dedent('''
{% extends 'base.html' %}
{% block title %}Servicios - Constructora Ejemplo{% endblock %}
{% block content %}
<h2>Servicios</h2>
<div class="grid">
{% for s in services %}
<div class="card"><h3>{{ s.title }}</h3><p>{{ s.desc }}</p></div>
{% endfor %}
</div>
{% endblock %}
'''),

    'proyectos.html': textwrap.dedent('''
{% extends 'base.html' %}
{% block title %}Proyectos - Constructora Ejemplo{% endblock %}
{% block content %}
<h2>Proyectos</h2>
<ul>{% for p in projects %}<li><strong>{{ p.name }}</strong> — {{ p.summary }}</li>{% endfor %}</ul>
{% endblock %}
'''),

    'contacto.html': textwrap.dedent('''
{% extends 'base.html' %}
{% block title %}Contacto - Constructora Ejemplo{% endblock %}
{% block content %}
<h2>Contacto</h2>
<div class="thankyou" style="display:none;background:#ecfdf5;color:#065f46;padding:12px;border-radius:6px;margin-bottom:12px;">¡Gracias! Hemos recibido tu mensaje.</div>
<form method="post" class="contact-form">
<label>Nombre<br><input name="name" required></label><br>
<label>Email<br><input type="email" name="email" required></label><br>
<label>Mensaje<br><textarea name="message" rows="5" required></textarea></label><br>
<button class="btn" type="submit">Enviar</button>
</form>
<script>
if(window.location.hash === '#gracias'){
  const el = document.querySelector('.thankyou')
  if(el) el.style.display = 'block'
}
</script>
{% endblock %}
''')
}

# ======================== CSS ========================
CSS = textwrap.dedent('''
:root{--max:1100px;font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,Helvetica,Arial;}
*{box-sizing:border-box}body{margin:0;color:#222;line-height:1.5}
.container{max-width:var(--max);margin:0 auto;padding:1rem;width:100%;}
.site-header{background:#0f172a;color:#fff;padding:.8rem 0}
.header-inner{display:flex;align-items:center;justify-content:space-between;gap:1rem}
.brand a{color:white;text-decoration:none;font-weight:700}
.main-nav a{color:#d1d5db;margin-left:1rem;text-decoration:none}
.hero{background:#e6eef8;padding:2rem;border-radius:8px;margin:1rem 0;text-align:center}
.btn{display:inline-block;padding:.6rem 1rem;border-radius:6px;background:#0ea5a4;color:white;text-decoration:none}
.grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:1rem}
.card{background:#fff;padding:1rem;border-radius:8px;box-shadow:0 2px 6px rgba(0,0,0,.06)}
.site-footer{padding:1rem 0;margin-top:2rem;border-top:1px solid #eee;text-align:center}
.contact-form input, .contact-form textarea{width:100%;padding:.6rem;margin:.3rem 0;border:1px solid #ddd;border-radius:6px}
@media (max-width:600px){.header-inner{flex-direction:column;align-items:flex-start}
.main-nav a{margin-left:0;display:inline-block;margin-right:0.6rem}}
''')

# ======================== FUNCIONES ========================
def ensure_path(p: Path):
    if not p.exists():
        p.mkdir(parents=True, exist_ok=True)

def write_file(path: Path, content: str, mode: str = 'w'):
    with open(path, mode, encoding='utf-8') as f:
        f.write(content)

def main():
    print(f"Preparando proyecto en: {BASE}")

    app_py_path = BASE / 'app.py'
    if not app_py_path.exists():
        write_file(app_py_path, APP_PY)
        print(f"Creado: {app_py_path}")
    else:
        print(f"Ya existe: {app_py_path}")

    templates_dir = BASE / 'templates'
    ensure_path(templates_dir)
    for name, txt in TEMPLATES.items():
        p = templates_dir / name
        if not p.exists():
            write_file(p, txt)
            print(f"Creado template: {p}")

    css_dir = BASE / 'static' / 'css'
    ensure_path(css_dir)
    css_path = css_dir / 'style.css'
    if not css_path.exists():
        write_file(css_path, CSS)
        print(f"Creado CSS: {css_path}")

    contactos = BASE / 'contactos.txt'
    if not contactos.exists():
        write_file(contactos, "# Lista de contactos enviados desde el formulario\\n")
        print(f"Creado archivo de contactos: {contactos}")

    print("\\nListo. Para arrancar la aplicación ejecuta:")
    print(" python app.py")

if __name__ == '__main__':
    main()





