<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Calculadora de Doses - Intubação e Sedação</title>
  <style>
    body { font-family: Arial, sans-serif; max-width: 800px; margin: auto; padding: 20px; }
    label, input, button, select { display: block; margin: 10px 0; }
    input, select { width: 100%; padding: 8px; }
    button { padding: 10px; background-color: #007BFF; color: white; border: none; cursor: pointer; }
    button:hover { background-color: #0056b3; }
    .result { background: #f8f9fa; padding: 10px; border-radius: 5px; margin-top: 20px; }
    ul { padding-left: 20px; }
    .tab { display: none; }
    .tab.active { display: block; }
    .tabs button { margin-right: 10px; }
    .copy-btn { margin-top: 5px; background-color: #28a745; border: none; padding: 5px; color: white; cursor: pointer; }
  </style>
</head>

  <img src="Captura de Tela 2023-04-13 às 21.48.51.png" alt="Logo Emerg" style="max-width: 100%; height: auto; margin-bottom: 20px;">
  <h2>Calculadora de Doses - Intubação e Sedação</h2>
  <label for="peso">Peso do paciente (kg):</label>
  <input type="number" id="peso" placeholder="Digite o peso em kg">

  <label for="intensidade">Selecione intensidade da dose:</label>
  <select id="intensidade">
    <option value="min">Dose mínima</option>
    <option value="padrao" selected>Dose padrão</option>
    <option value="max">Dose máxima</option>
  </select>

  <div class="tabs">
    <button onclick="mostrarAba('indu')">Indução e Sedação</button>
    <button onclick="mostrarAba('bloqueio')">Bloqueio Neuromuscular</button>
    <button onclick="mostrarAba('sedacao_cont')">Sedação Contínua</button>
    <button onclick="mostrarAba('vasoativas')">Drogas Vasoativas</button>
  </div>

  <button onclick="calcularDoses()">Calcular Doses</button>

  <div id="resultados" class="result"></div>

  <script>
    function mostrarAba(aba) {
      const abas = document.querySelectorAll('.tab');
      abas.forEach(tab => tab.classList.remove('active'));
      document.getElementById(aba).classList.add('active');
    }

    function copiarTexto(texto) {
      navigator.clipboard.writeText(texto);
      alert("Texto copiado!");
    }

    function calcularDoses() {
      const peso = parseFloat(document.getElementById("peso").value);
      const intensidade = document.getElementById("intensidade").value;

      if (isNaN(peso) || peso <= 0) {
        alert("Por favor, insira um peso válido.");
        return;
      }

      const mult = intensidade === "min" ? 0.75 : intensidade === "max" ? 1.25 : 1.0;

      const categorias = {
        indu: [
          { nome: "Etomidato", dose: 0.3, unidade: "mg/kg", indicacao: "Indução em pacientes instáveis", ref: "UpToDate" },
          { nome: "Quetamina (indução)", dose: 2, unidade: "mg/kg", indicacao: "Broncoespasmo, instabilidade", ref: "Manual do ATLS" },
          { nome: "Propofol (indução)", dose: 2, unidade: "mg/kg", indicacao: "Pacientes estáveis", ref: "UpToDate" },
          { nome: "Fentanil (indução)", dose: 2, unidade: "mcg/kg", indicacao: "Atenuação da resposta autonômica", ref: "Manual de Anestesia" },
          { nome: "Quetamina (sedação)", dose: 0.5, unidade: "mg/kg", indicacao: "Sedação em procedimentos dolorosos", ref: "Tintinalli" },
          { nome: "Propofol (sedação)", dose: 0.5, unidade: "mg/kg", indicacao: "Procedimentos rápidos", ref: "Manual de Anestesia" },
          { nome: "Fentanil (sedação)", dose: 1, unidade: "mcg/kg", indicacao: "Sedação ventilatória", ref: "UpToDate" }
        ],
        bloqueio: [
          { nome: "Succinilcolina", dose: 1.5, unidade: "mg/kg", indicacao: "Paralisia rápida e curta", ref: "Manual do ATLS" },
          { nome: "Rocurônio", dose: 1.2, unidade: "mg/kg", indicacao: "Bloqueio prolongado", ref: "UpToDate" }
        ],
        sedacao_cont: [
          { nome: "Fentanil (manutenção)", dose: 1.5, unidade: "mcg/kg/h", indicacao: "Analgesia contínua", ref: "UpToDate" },
          { nome: "Propofol (manutenção)", dose: 2, unidade: "mg/kg/h", indicacao: "Sedação em UTI", ref: "UpToDate" },
          { nome: "Midazolam", dose: 0.05, unidade: "mg/kg/h", indicacao: "Sedação contínua (alternativa)", ref: "UpToDate" }
        ],
        vasoativas: [
          { nome: "Noradrenalina", dose: 0.05, unidade: "mcg/kg/min", indicacao: "Choque vasodilatado", ref: "Diretrizes de Sepse" },
          { nome: "Dobutamina", dose: 5, unidade: "mcg/kg/min", indicacao: "Disfunção sistólica", ref: "UpToDate" },
          { nome: "Vasopressina", dose: 0.03, unidade: "U/min", indicacao: "Choque refratário", ref: "UpToDate" },
          { nome: "Adrenalina", dose: 0.05, unidade: "mcg/kg/min", indicacao: "Choque anafilático ou refratário", ref: "UpToDate" }
        ]
      };

      let html = "";
      for (let categoria in categorias) {
        html += `<div id="${categoria}" class="tab">
          <h3>${categoria.toUpperCase().replace("_", " ")}</h3>
          <ul>`;
        categorias[categoria].forEach(d => {
          const total = (peso * d.dose * mult).toFixed(2);
          const texto = `${d.nome}: ${total} ${d.unidade}`;
          html += `<li><strong>${d.nome}:</strong> ${total} ${d.unidade}<br>
                    <em>Indicação:</em> ${d.indicacao}<br>
                    <em>Fonte:</em> ${d.ref}<br>
                    <button class='copy-btn' onclick="copiarTexto('${texto}')">Copiar</button>
                  </li>`;
        });
        html += `</ul>`;

        if (categoria === "indu") {
          html += `
            <h4>Descrição da Intubação:</h4>
            <p><strong>VIA AÉREA ANATOMICAMENTE DIFÍCIL:</strong><br>
            VIA AÉREA FISIOLOGICAMENTE DIFÍCIL:<br>
            MEDICAÇÕES:<br>
            DISPOSITIVOS:<br>
            CL: grau<br>
            DESCRIÇÃO:<br>
            PASSADO TUBO:<br>
            PARÂMETROS VM: MODO PVC - Pins PEEP FR VC FIO2:<br>
            SINAIS VITAIS 5 MINUTOS APÓS: PA: SAT: PA: FC:<br>
            SEDAÇÃO CONTÍNUA: MIDAZOLAM (5 AMPOLAS EM 200ML DE SF0,9%) + FENTANIL (5 AMPOLAS EM 200ML DE SF0,9%) EM INFUSÃO CONTÍNUA EM BOMBA.</p>`;
        }

        html += `</div>`;
      }

      document.getElementById("resultados").innerHTML = html;
      mostrarAba('indu');
    }
  </script>
</body>
</html>
