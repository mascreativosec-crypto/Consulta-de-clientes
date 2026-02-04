 <!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Consulta de Cliente - Eureka Education</title>
    <style>
        *{margin:0;padding:0;box-sizing:border-box}
        body{
            font-family:'Segoe UI',Tahoma,Geneva,Verdana,sans-serif;
            background:linear-gradient(135deg,#667eea 0%,#764ba2 100%);
            min-height:100vh;
            padding:20px
        }
        .container{
            max-width:900px;
            margin:20px auto;
            background:#fff;
            border-radius:20px;
            box-shadow:0 20px 60px rgba(0,0,0,.3);
            padding:40px
        }
        h1{text-align:center;color:#333;margin-bottom:10px}
        .subtitle{text-align:center;color:#666;margin-bottom:30px;font-size:14px}

        .search-options{
            background:#f8f9fa;
            padding:20px;
            border-radius:10px;
            border:2px solid #e9ecef;
            margin-bottom:25px
        }
        .search-options h3{color:#667eea;margin-bottom:15px}

        .radio-group{
            display:grid;
            grid-template-columns:repeat(auto-fit,minmax(150px,1fr));
            gap:15px
        }
        .radio-option{
            background:#fff;
            padding:12px;
            border-radius:8px;
            border:2px solid #e0e0e0;
            display:flex;
            gap:10px;
            align-items:center;
            cursor:pointer
        }
        .radio-option.active{background:#667eea;border-color:#667eea}
        .radio-option.active label{color:#fff}

        .form-group{margin-bottom:25px}
        label{font-weight:600;margin-bottom:8px;display:block}
        input{
            width:100%;
            padding:12px;
            border-radius:10px;
            border:2px solid #e0e0e0;
            font-size:16px
        }

        button{
            width:100%;
            padding:15px;
            border:none;
            border-radius:10px;
            background:linear-gradient(135deg,#667eea,#764ba2);
            color:#fff;
            font-size:16px;
            font-weight:600;
            cursor:pointer
        }

        .loading{text-align:center;display:none;margin:20px 0}
        .loading.active{display:block}

        .spinner{
            width:40px;height:40px;
            border:4px solid #f3f3f3;
            border-top:4px solid #667eea;
            border-radius:50%;
            animation:spin 1s linear infinite;
            margin:auto
        }
        @keyframes spin{to{transform:rotate(360deg)}}

        .hidden{display:none}
        .result{display:none;margin-top:30px}
        .result.active{display:block}

        .info-card{
            background:#f8f9fa;
            padding:20px;
            border-radius:10px;
            margin-bottom:20px
        }
        .info-row{
            display:flex;
            justify-content:space-between;
            padding:10px 0;
            border-bottom:1px solid #ddd
        }
        .info-row:last-child{border-bottom:none}

        .section-title{
            color:#667eea;
            font-weight:700;
            margin-bottom:10px;
            border-bottom:2px solid #667eea
        }

        .mensualidades-grid{
            display:grid;
            grid-template-columns:repeat(auto-fill,minmax(140px,1fr));
            gap:10px
        }
        .mensualidad-item{
            background:#fff;
            border:2px solid #e0e0e0;
            padding:10px;
            border-radius:8px;
            text-align:center;
            cursor:pointer
        }
        .mensualidad-item.selected{
            background:#667eea;
            color:#fff;
            border-color:#667eea
        }

        .error{background:#f8d7da;color:#721c24;padding:15px;border-radius:10px;margin-top:20px}
        .success{background:#d4edda;color:#155724;padding:15px;border-radius:10px;margin-top:20px}
        .btn-secondary{background:#6c757d;margin-top:10px}
    </style>
</head>

<body>
<div class="container">
    <h1>🔍 Consulta de Cliente</h1>
    <p class="subtitle">Sistema de Gestión de Pagos - Eureka Education</p>

    <div id="consultaForm">
        <form id="searchForm">
            <div class="search-options">
                <h3>📋 Seleccione el tipo de búsqueda</h3>
                <div class="radio-group">
                    <div class="radio-option active">
                        <input type="radio" name="tipoBusqueda" value="contrato" checked>
                        <label>Número de Contrato</label>
                    </div>
                    <div class="radio-option">
                        <input type="radio" name="tipoBusqueda" value="nombre">
                        <label>Nombre del Titular</label>
                    </div>
                    <div class="radio-option">
                        <input type="radio" name="tipoBusqueda" value="cedula">
                        <label>Cédula</label>
                    </div>
                </div>
            </div>

            <div class="form-group" id="grupoContrato">
                <label>Número de Contrato</label>
                <input id="numeroContrato">
            </div>

            <div class="form-group hidden" id="grupoNombre">
                <label>Nombre del Titular</label>
                <input id="nombreTitular">
            </div>

            <div class="form-group hidden" id="grupoCedula">
                <label>Cédula</label>
                <input id="cedula">
            </div>

            <button>🔍 Consultar Cliente</button>
        </form>
    </div>

    <div class="loading" id="loading">
        <div class="spinner"></div>
        <p>Buscando cliente…</p>
    </div>

    <div class="result" id="result"></div>
    <div id="mensajes"></div>
</div>

<script>
const WEBHOOK_URL = 'http://localhost:5678/webhook-test/webhook-pago';

const form = document.getElementById('searchForm');
const loading = document.getElementById('loading');
const mensajes = document.getElementById('mensajes');

form.addEventListener('submit', async e => {
    e.preventDefault();

    const tipo = document.querySelector('input[name="tipoBusqueda"]:checked').value;
    let valor = '';

    if (tipo === 'contrato') valor = numeroContrato.value;
    if (tipo === 'nombre') valor = nombreTitular.value;
    if (tipo === 'cedula') valor = cedula.value;

    if (!valor) {
        mensajes.innerHTML = '<div class="error">Ingrese un valor</div>';
        return;
    }

    loading.classList.add('active');
    mensajes.innerHTML = '';

    try {
        const res = await fetch(WEBHOOK_URL,{
            method:'POST',
            headers:{'Content-Type':'application/json'},
            body:JSON.stringify({
                accion:'consultar',
                tipoBusqueda:tipo,
                valorBusqueda:valor
            })
        });

        const data = await res.json();
        loading.classList.remove('active');

        mensajes.innerHTML = data.success
            ? '<div class="success">Consulta enviada a n8n correctamente</div>'
            : '<div class="error">'+data.mensaje+'</div>';

    } catch {
        loading.classList.remove('active');
        mensajes.innerHTML = '<div class="error">Error de conexión con n8n</div>';
    }
});
</script>
</body>
</html>
