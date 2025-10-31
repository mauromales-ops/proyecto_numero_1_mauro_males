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
        line = f"{timestamp} | {name} | {email} | {body}\n"
        with open('contactos.txt', 'a', encoding='utf-8') as f:
            f.write(line)
        return redirect(url_for('contacto') + '#gracias')
    return render_template("contacto.html", meta=meta)

@app.route('/robots.txt')
def robots_txt():
    host = request.url_root.rstrip('/')
    txt = f"User-agent: *\nAllow: /\nSitemap: {host}/sitemap.xml\n"
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
    sitemap_xml = "<?xml version='1.0' encoding='UTF-8'?>\n" \
                  "<urlset xmlns='http://www.sitemaps.org/schemas/sitemap/0.9'>\n" \
                  + "\n".join(xml_items) + "\n</urlset>"
    return Response(sitemap_xml, mimetype='application/xml')

if __name__ == '__main__':
    port = int(os.environ.get('PORT', 8000))
    app.run(host='0.0.0.0', port=port)

