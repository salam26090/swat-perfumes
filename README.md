rava-parfume/
├── app.py
├── requirements.txt
├── Procfile
├── render.yaml
├── .gitignore
├── runtime.txt
├── templates/
│   ├── base.html
│   ├── index.html
│   └── success.html
└── static/
    ├── css/style.css
    └── js/script.js
import os
from urllib.parse import quote
from flask import Flask, render_template, request

app = Flask(__name__)
app.secret_key = os.environ.get("SECRET_KEY", "rava-parfume-secret-key")

SHOP_INFO = {
    "name": "RAVA PARFUME",
    "brand": "RAVA PARFUME",
    "currency": "PKR",
    "whatsapp": "00923454828100",
    "whatsapp_url": "923454828100",
    "email": "salam26090@gmail.com"
}

COLLECTIONS = {
    "Luxury": {
        "name": "Luxury Collection",
        "perfumes": [
            {
                "id": "rava01",
                "name": "Rava Royal Oud",
                "desc": "A rich and luxurious fragrance.",
                "prices": {"10ml": 500, "20ml": 900, "50ml": 1800, "100ml": 3000}
            },
            {
                "id": "rava02",
                "name": "Rava Golden Amber",
                "desc": "A warm and elegant fragrance.",
                "prices": {"10ml": 500, "20ml": 900, "50ml": 1800, "100ml": 3000}
            },
            {
                "id": "rava03",
                "name": "Rava Velvet Rose",
                "desc": "A soft and beautiful floral fragrance.",
                "prices": {"10ml": 450, "20ml": 850, "50ml": 1700, "100ml": 2800}
            }
        ]
    }
}


@app.route("/")
def home():
    return render_template("index.html", shop=SHOP_INFO, collections=COLLECTIONS)


@app.route("/checkout", methods=["POST"])
def checkout():
    perfume_name  = request.form.get("perfume_name", "").strip()
    size          = request.form.get("size", "").strip()
    price         = request.form.get("price", "").strip()
    customer_name = request.form.get("customer_name", "").strip()
    phone         = request.form.get("phone", "").strip()
    address       = request.form.get("address", "").strip()

    missing = []
    if not perfume_name: missing.append("Perfume")
    if not size:         missing.append("Size")
    if not customer_name: missing.append("Name")
    if not phone:        missing.append("Phone")
    if not address:      missing.append("Address")

    if missing:
        return render_template(
            "index.html",
            shop=SHOP_INFO,
            collections=COLLECTIONS,
            error=f"براہ کرم یہ فیلڈز بھریں: {', '.join(missing)}"
        ), 400

    message = (
        f"*New RAVA PARFUME Order*\n"
        f"──────────────\n"
        f"🌸 Perfume: {perfume_name}\n"
        f"📦 Size: {size}\n"
        f"💰 Price: PKR {price}\n"
        f"──────────────\n"
        f"👤 Customer: {customer_name}\n"
        f"📞 Phone: {phone}\n"
        f"📍 Address: {address}"
    )

    whatsapp_url = f"https://wa.me/{SHOP_INFO['whatsapp_url']}?text={quote(message)}"

    return render_template(
        "success.html",
        shop=SHOP_INFO,
        whatsapp_url=whatsapp_url,
        perfume_name=perfume_name,
        size=size,
        price=price,
        customer_name=customer_name
    )


@app.route("/health")
def health():
    return {"status": "ok"}, 200


if __name__ == "__main__":
    port = int(os.environ.get("PORT", 5000))
    app.run(host="0.0.0.0", port=port, debug=False)
Flask==3.0.3
gunicorn==22.0.0

web: gunicorn app:app
services:
  - type: web
    name: rava-parfume
    env: python
    region: singapore
    buildCommand: pip install -r requirements.txt
    startCommand: gunicorn app:app
    plan: free
    autoDeploy: true

__pycache__/
*.pyc
.env
venv/
.venv/
*.log
.DS_Store
python-3.11.9
<!DOCTYPE html>
<html lang="ur" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="sviewport" content="width=device-width, initial-scale=1app.0">
    <title>{% block title }}</ %}{{ shop.name }}{%p endblock %}</title>
    <>
link href="https://fonts.googleapis.com/css2?family=Noto+Nastaliq+Urdu:wght@400;700&family=Poppins:wght@300;400;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
    <header class="site-header">
        <div class="container">
            <div class="logo">
                <h1>{{ shop.name }}</h1>
                <p>{{ shop.brand }}</p>
            </div>
            <div class="contact">
                <a href="https://wa.me/{{ shop.whatsapp_url }}" target="_blank">📞 {{ shop.whatsapp }}</a>
                <a href="mailto:{{ shop.email }}">✉️ {{ shop.email }}</a>
            </div>
        </div>
    </header>

    <main>{% block content %}{% endblock %}</main>

    <footer class="site-footer">
        <div class="container">
            <p>© 2025 {{ shop.name }} — تمام حقوق محفوظ ہیں</p>
            <p>WhatsApp: {{ shop.what        </div>
    </footer>

    <script src="{{ url_for('static', filename='js/script.js') }}"></script>
</body>
</html>

{% extends 'base.html' %}
{% block title %}{{ shop.name }} — Luxury Fragrances{% endblock %}

{% block content %}
<section class="hero">
    <div class="container">
        <h1 class="hero-title">LUXURY FRAGRANCE COLLECTION</h1>
        <h2 class="hero-subtitle">{{ shop.name }}</h2>
        <p class="hero-text">{{ shop.brand }} اور منتخب خوشبوؤں کا شاندار مجموعہ</p>
        <a href="#collection" class="btn-primary">کلیکشن دیکھیں ↓</a>
    </div>
</section>

{% if error %}
<div class="container">
    <div class="alert-error">⚠️ {{ error }}</div>
</div>
{% endif %}

<section id="collection" class="collection-section">
    <div class="container">
        <h2 class="section-title">ہمارے پرفیوم</h2>

        {% for col_key, col in collections.items() %}
        <div class="collection-block">
            <h3 class="collection-name">{{ col.name }}</h3>
            <div class="products-grid">
                {% for p in col.perfumes %}
                <div class="product-card">
                    <div class="product-info">
                        <h4 class="product-name">{{ p.name }}</h4>
                        <p class="product-brand">{{ shop.brand }} · {{ col.name }}</p>
                        <p class="product-desc">{{ p.desc }}</p>
                    </div>

                    <form class="order-form" action="/checkout" method="POST" onsubmit="return prepareOrder(this);">
                        <input type="hidden" name="perfume_name" value="{{ p.name }}">

                        <label class="field-label">سائز</label>
                        <select name="size" class="form-input size-select" onchange="updatePrice(this);" required>
                            {% for size, price in p.prices.items() %}
                            <option value="{{ size }}" data-price="{{ price }}">
                                {{ size }} — PKR {{ "{:,}".format(price) }}
                            </option>
                            {% endfor %}
                        </select>

                        <p class="price-display">
                            قیمت: <strong>PKR <span class="price-value">{{ "{:,}".format(p.prices['10ml']) }}</span></strong>
                        </p>
                        <input type="hidden" name="price" class="price-input" value="{{ p.prices['10ml'] }}">

                        <label class="field-label">آپ کا نام</label>
                        <input type="text" name="customer_name" class="form-input" placeholder="مکمل نام" required>

                        <label class="field-label">موبائل نمبر</label>
                        <input type="tel" name="phone" class="form-input" placeholder="03xxxxxxxxx" required>

                        <label class="field-label">مکمل پتہ</label>
                        <textarea name="address" class="form-input" rows="3" placeholder="گھر نمبر، گلی، علاقہ، شہر" required></textarea>

                        <button type="submit" class="btn-order">واٹس ایپ پر آرڈر تیار کریں</button>
                    </form>
                </div>
                {% endfor %}
            </div>
        </div>
        {% endfor %}
    </div>
</section>
{% endblock %}

{% extends 'base.html' %}
{% block title %}آرڈر کی تصدیق — {{ shop.name }}{% endblock %}

{% block content %}
<section class="success-section">
    <div class="container">
        <div class="success-card">
            <div class="success-icon">✅</div>
            <h1>آپ کا آرڈر تیار ہے!</h1>
            <p class="success-text">
                شکریہ <strong>{{ customer_name }}</strong>! براہ کرم نیچے والے بٹن پر کلک کر کے واٹس ایپ پر آرڈر مکمل کریں۔
            </p>

            <div class="order-summary">
                <h3>آرڈر کی تفصیل:</h3>
                <ul>
                    <li><strong>پرفیوم:</strong> {{ perfume_name }}</li>
                    <li><strong>سائز:</strong> {{ size }}</li>
                    <li><strong>قیمت:</strong> PKR {{ price }}</li>
                </ul>
            </div>

            <a href="{{ whatsapp_url }}" target="_blank" class="btn-whatsapp">📲 واٹس ایپ پر آرڈر بھیجیں</a>
            <a href="/" class="btn-secondary">← مزید خریداری کریں</a>
        </div>
    </div>
</section>
{% endblock %}


* { margin: 0; padding: 0; box-sizing: border-box; }

body {
    font-family: 'Poppins', 'Noto Nastaliq Urdu', sans-serif;
    background: #faf8f5;
    color: #2c2c2c;
    line-height: 1.7;
}

.container { max-width: 1200px; margin: 0 auto; padding: 0 20px; }

.site-header {
    background: linear-gradient(135deg, #1a1a1a 0%, #2d1810 100%);
    color: #d4af37;
    padding: 20px 0;
    box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}
.site-header .container { display: flex; justify-content: space-between; align-items: center; flex-wrap: wrap; gap: 15px; }
.logo h1 { font-size: 1.8rem; font-weight: 700; letter-spacing: 2px; color: #d4af37; }
.logo p { font-size: 0.9rem; color: #c9a961; letter-spacing: 3px; }
.contact a { color: #d4af37; text-decoration: none; margin-inline-start: 15px; font-size: 0.95rem; }
.contact a:hover { color: #fff; }

.hero { background: linear-gradient(135deg, #2d1810 0%, #1a1a1a 100%); color: #fff; text-align: center; padding: 80px 20px; }
.hero-title { font-size: 2.5rem; letter-spacing: 4px; color: #d4af37; margin-bottom: 15px; }
.hero-subtitle { font-size: 2rem; margin-bottom: 10px; }
.hero-text { font-size: 1.1rem; color: #ddd; margin-bottom: 30px; font-family: 'Noto Nastaliq Urdu', serif; }
.btn-primary { display: inline-block; background: #d4af37; color: #1a1a1a; padding: 14px 35px; border-radius: 50px; text-decoration: none; font-weight: 600; letter-spacing: 1px; transition: all 0.3s; }
.btn-primary:hover { background: #fff; transform: translateY(-2px); box-shadow: 0 10px 25px rgba(212,175,55,0.4); }

.collection-section { padding: 60px 0; }
.section-title { text-align: center; font-size: 2rem; color: #2d1810; margin-bottom: 40px; font-family: 'Noto Nastaliq Urdu', serif; position: relative; }
.section-title::after { content: ''; display: block; width: 80px; height: 3px; background: #d4af37; margin: 15px auto 0; }
.collection-block { margin-bottom: 50px; }
.collection-name { font-size: 1.5rem; color: #2d1810; margin-bottom: 25px; text-align: center; }
.products-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(320px, 1fr)); gap: 30px; }

.product-card { background: #fff; border-radius: 15px; overflow: hidden; box-shadow: 0 5px 25px rgba(0,0,0,0.08); transition: transform 0.3s, box-shadow 0.3s; border: 1px solid #f0e6d2; }
.product-card:hover { transform: translateY(-5px); box-shadow: 0 15px 40px rgba(212,175,55,0.2); }
.product-info { padding: 25px 25px 15px; background: linear-gradient(135deg, #2d1810 0%, #1a1a1a 100%); color: #fff; text-align: center; }
.product-name { font-size: 1.3rem; color: #d4af37; margin-bottom: 8px; }
.product-brand { font-size: 0.85rem; color: #c9a961; letter-spacing: 1px; margin-bottom: 12px; }
.product-desc { font-size: 0.95rem; color: #ddd; font-family: 'Noto Nastaliq Urdu', serif; }

.order-form { padding: 25px; }
.field-label { display: block; font-size: 0.9rem; color: #2d1810; font-weight: 600; margin: 12px 0 6px; font-family: 'Noto Nastaliq Urdu', serif; }
.form-input { width: 100%; padding: 12px 15px; border: 2px solid #f0e6d2; border-radius: 10px; font-size: 1rem; font-family: inherit; background: #faf8f5; transition: border-color 0.3s; }
.form-input:focus { outline: none; border-color: #d4af37; background: #fff; }
.price-display { margin: 15px 0; font-size: 1.1rem; color: #2d1810; text-align: center; font-family: 'Noto Nastaliq Urdu', serif; }
.price-display strong { color: #d4af37; font-size: 1.3rem; }
.btn-order { width: 100%; background: #25D366; color: #fff; padding: 15px; border: none; border-radius: 10px; font-size: 1rem; font-weight: 600; cursor: pointer; margin-top: 20px; font-family: 'Noto Nastaliq Urdu', serif; transition: all 0.3s; }
.btn-order:hover { background: #1da851; transform: translateY(-2px); box-shadow: 0 8px 20px rgba(37,211,102,0.3); }

.alert-error { background: #fee; color: #c00; padding: 15px 20px; border-radius: 10px; margin: 20px 0; border-right: 4px solid #c00; font-family: 'Noto Nastaliq Urdu', serif; }

.success-section { padding: 60px 0; min-height: 70vh; display: flex; align-items: center; }
.success-card { background: #fff; max-width: 600px; margin: 0 auto; padding: 50px 40px; border-radius: 20px; text-align: center; box-shadow: 0 15px 50px rgba(0,0,0,0.1); border-top: 5px solid #d4af37; }
.success-icon { font-size: 4rem; margin-bottom: 20px; }
.success-card h1 { color: #2d1810; margin-bottom: 15px; font-family: 'Noto Nastaliq Urdu', serif; }
.success-text { color: #666; margin-bottom: 25px; font-family: 'Noto Nastaliq Urdu', serif; }
.order-summary { background: #faf8f5; padding: 20px; border-radius: 12px; text-align: right; margin: 25px 0; border: 1px dashed #d4af37; }
.order-summary h3 { color: #2d1810; margin-bottom: 12px; font-size: 1.1rem; }
.order-summary ul { list-style: none; }
.order-summary li { padding: 6px 0; color: #444; }
.btn-whatsapp { display: inline-block; background: #25D366; color: #fff; padding: 15px 40px; border-radius: 50px; text-decoration: none; font-weight: 600; margin: 10px 5px; transition: all 0.3s; }
.btn-whatsapp:hover { background: #1da851; transform: translateY(-2px); box-shadow: 0 8px 20px rgba(37,211,102,0.3); }
.btn-secondary { display: inline-block; color: #2d1810; padding: 15px 30px; text-decoration: none; margin: 10px 5px; font-family: 'Noto Nastaliq Urdu', serif; }
.btn-secondary:hover { color: #d4af37; }

.site-footer { background: #1a1a1a; color: #888; text-align: center; padding: 25px 0; margin-top: 50px; }
.site-footer p { margin: 5px 0; font-size: 0.9rem; }

@media (max-width: 600px) {
    .hero-title { font-size: 1.6rem; letter-spacing: 2px; }
    .hero-subtitle { font-size: 1.4rem; }
    .site-header .container { flex-direction: column; text-align: center; }
    .contact a { display: block; margin: 5px 0; }
    .success-card { padding: 35px 20px; }
}

function updatePrice(selectEl) {
    const selectedOption = selectEl.options[selectEl.selectedIndex];
    const price = selectedOption.getAttribute('data-price');
    const form = selectEl.closest('form');
    const priceValue = form.querySelector('.price-value');
    const priceInput = form.querySelector('.price-input');
    if (price) {
        priceValue.textContent = Number(price).toLocaleString();
        priceInput.value = price;
    }
}

function prepareOrder(form) {
    const name = form.querySelector('[name="customer_name"]').value.trim();
    const phone = form.querySelector('[name="phone"]').value.trim();
    const address = form.querySelector('[name="address"]').value.trim();
    if (!name || !phone || !address) {
        alert('براہ کرم تمام فیلڈز بھریں');
        return false;
    }
    if (phone.length < 10) {
        alert('براہ کرم درست موبائل نمبر درج کریں');
        return false;
    }
    return true;
}

document.addEventListener('DOMContentLoaded', () => {
    document.querySelectorAll('.size-select').forEach(select => updatePrice(select));
});

