<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Painel RPG Dinâmico</title>
    <style>
        :root {
            --bg: #0d1117; --card: #161b22; --primary: #58a6ff; --text: #c9d1d9; --accent: #238636;
        }
        body { font-family: 'Segoe UI', sans-serif; background: var(--bg); color: var(--text); margin: 0; padding: 15px; display: flex; justify-content: center; }
        .wrapper { width: 100%; max-width: 500px; background: var(--card); border-radius: 12px; padding: 20px; box-shadow: 0 8px 32px rgba(0,0,0,0.5); border: 1px solid #30363d; }
        
        h2 { text-align: center; color: var(--primary); margin-bottom: 20px; text-transform: uppercase; letter-spacing: 2px; }
        
        .field-group { margin-bottom: 15px; }
        label { display: block; font-size: 0.75em; color: #8b949e; margin-bottom: 5px; font-weight: bold; }
        input, select { width: 100%; padding: 12px; background: #010409; border: 1px solid #30363d; color: white; border-radius: 6px; box-sizing: border-box; font-size: 1em; }
        
        .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin: 20px 0; }
        .stat-box { background: #0d1117; padding: 15px; border-radius: 8px; border: 1px solid #30363d; text-align: center; }
        .stat-box.total { border-color: var(--accent); }
        .stat-num { font-size: 1.5em; font-weight: bold; display: block; }
        .stat-label { font-size: 0.7em; color: #8b949e; text-transform: uppercase; }

        .equip-table { width: 100%; margin-top: 20px; border-collapse: collapse; }
        .equip-table th { font-size: 0.7em; color: #8b949e; text-align: left; padding: 5px; }
        .equip-row input { padding: 5px; font-size: 0.8em; margin-bottom: 5px; }

        .skill-card { background: #0d1117; padding: 12px; border-radius: 8px; margin-top: 10px; border-left: 4px solid var(--primary); }
        .skill-name { font-weight: bold; color: var(--primary); font-size: 0.9em; }
        .skill-desc { font-size: 0.85em; color: #8b949e; margin-top: 5px; line-height: 1.4; }
        .skill-lvl { float: right; font-size: 0.7em; background: #21262d; padding: 2px 5px; border-radius: 4px; }

        .xp-bar-container { background: #30363d; height: 8px; border-radius: 4px; margin-top: 10px; overflow: hidden; }
        .xp-bar-fill { background: var(--accent); height: 100%; width: 0%; transition: width 0.4s; }

        .section-title { border-bottom: 1px solid #30363d; margin: 25px 0 10px; padding-bottom: 5px; color: var(--primary); font-size: 0.9em; font-weight: bold; }
        .btn-reset { background: #f85149; color: white; border: none; padding: 10px; width: 100%; border-radius: 6px; cursor: pointer; margin-top: 20px; font-weight: bold; opacity: 0.8; }
    </style>
</head>
<body>

<div class="wrapper">
    <h2>🛡️ Ficha de Batalha</h2>

    <div class="field-group">
        <label>🌀 NOME DO PARTICIPANTE</label>
        <input type="text" id="part-name" placeholder="Digite seu nome..." oninput="salvar()">
    </div>

    <div class="field-group">
        <label>😎 PERSONAGEM</label>
        <select id="char-select" onchange="salvar()">
            <option value="DRUZION">DRUZION (Curandeiro)</option>
            <option value="GUILBERT">GUILBERT (Arqueiro)</option>
            <option value="MELKYOR">MELKYOR (Mago)</option>
            <option value="SCAR">SCAR (Guerreiro)</option>
            <option value="DEWPEE">DEWPEE (Pistoleiro)</option>
        </select>
    </div>

    <div class="stats-grid">
        <div class="field-group" style="margin:0">
            <label>🔺 NÍVEL</label>
            <input type="number" id="lvl" value="1" min="1" oninput="salvar()">
        </div>
        <div class="field-group" style="margin:0">
            <label>✨ XP ATUAL</label>
            <input type="number" id="xp" value="0" oninput="salvar()">
        </div>
    </div>
    <div id="xp-text" style="font-size: 0.7em; text-align: right; color: #8b949e;">XP para Nível 2: 500</div>
    <div class="xp-bar-container"><div class="xp-bar-fill" id="xp-fill"></div></div>

    <div class="section-title">📊 ATRIBUTOS TOTAIS (BASE + EQUIP)</div>
    <div class="stats-grid">
        <div class="stat-box total">
            <span class="stat-label">💖 Vida Total</span>
            <span class="stat-num" id="total-hp">0</span>
        </div>
        <div class="stat-box total">
            <span class="stat-label">👊🏽 Dano Total</span>
            <span class="stat-num" id="total-dmg">0</span>
        </div>
        <div class="stat-box">
            <span class="stat-label">🛡️ Escudo</span>
            <span class="stat-num" id="total-shd">0</span>
        </div>
        <div class="stat-box">
            <span class="stat-label">💰 Ouro</span>
            <input type="number" id="ouro" value="0" style="padding:2px; text-align:center;" oninput="salvar()">
        </div>
    </div>

    <div class="section-title">🔶 EQUIPAMENTOS</div>
    <table class="equip-table">
        <thead><tr><th>NOME DO ITEM</th><th>+HP</th><th>+DMG</th><th>+SHD</th></tr></thead>
        <tbody id="equip-list">
            <!-- Gerado via JS para facilitar salvar -->
        </tbody>
    </table>

    <div class="section-title">💫 HABILIDADES DESBLOQUEADAS</div>
    <div id="skills-container"></div>

    <button class="btn-reset" onclick="resetar()">LIMPAR FICHA / RESETAR</button>
</div>

<script>
    // === DADOS MESTRES ===
    const data = {
        chars: {
            DRUZION: { hp: 90, dmg: 4, class: "Curandeiro", p: "AURA VERDE", pd: "Ao curar um aliado, recupera vida dele igual ao seu dano total." },
            GUILBERT: { hp: 80, dmg: 8, class: "Arqueiro", p: "RESPEITO", pd: "Se houver 2 aliados ou menos, você não pode ser alvo de ataques." },
            MELKYOR: { hp: 70, dmg: 12, class: "Mago", p: "MANAMUNDO", pd: "Causa +10% de dano para cada aliado na batalha." },
            SCAR: { hp: 60, dmg: 16, class: "Guerreiro", p: "CONTRAGOLPE", pd: "Ao receber dano par, causa 20% do dano ao atacante." },
            DEWPEE: { hp: 50, dmg: 20, class: "Pistoleiro", p: "MUNIÇÃO EXPLOSIVA", pd: "Causa 20% do dano a todos os aliados do oponente." }
        },
        skills: [
            { lvl: 1, type: "Ativa 1", DRUZION: "ALVO", GUILBERT: "CHUVA DE FLECHAS", MELKYOR: "VENENO MÁGICO", SCAR: "LÂMINA VAMPÍRICA", DEWPEE: "RECARREGAR" },
            { lvl: 1, type: "Extra", DRUZION: "REPOUSO", GUILBERT: "", MELKYOR: "", SCAR: "", DEWPEE: "" },
            { lvl: 4, type: "Ativa 2", DRUZION: "CURA FRACA", GUILBERT: "PRISMA ARCANO", MELKYOR: "RECARGA DE ENERGIA", SCAR: "UNIÃO AMISTOSA", DEWPEE: "TIRO DUPLO" },
            { lvl: 7, type: "Ativa 3", DRUZION: "CONFUSÃO", GUILBERT: "GRITO DE GUERRA", MELKYOR: "CORROSÃO E DRENAGEM", SCAR: "MALDIÇÃO", DEWPEE: "BALA DE PRECISÃO" },
            { lvl: 10, type: "Ativa 4", DRUZION: "DUETO", GUILBERT: "LÂMINA GIRATÓRIA", MELKYOR: "DUALIDADE", SCAR: "ARCO-ÍRIS", DEWPEE: "RAJADA FRENÉTICA" }
            // O sistema aceita qualquer nível aqui seguindo este padrão
        ]
    };

    // Criar campos de equipamentos
    const equipBody = document.getElementById('equip-list');
    for(let i=1; i<=4; i++) {
        equipBody.innerHTML += `
            <tr class="equip-row">
                <td><input type="text" id="eq_n_${i}" oninput="salvar()"></td>
                <td><input type="number" id="eq_h_${i}" oninput="salvar()"></td>
                <td><input type="number" id="eq_d_${i}" oninput="salvar()"></td>
                <td><input type="number" id="eq_s_${i}" oninput="salvar()"></td>
            </tr>`;
    }

    function salvar() {
        const estado = {
            part: document.getElementById('part-name').value,
            char: document.getElementById('char-select').value,
            lvl: parseInt(document.getElementById('lvl').value) || 1,
            xp: parseInt(document.getElementById('xp').value) || 0,
            ouro: document.getElementById('ouro').value,
            equips: []
        };
        for(let i=1; i<=4; i++) {
            estado.equips.push({
                h: parseInt(document.getElementById(`eq_h_${i}`).value) || 0,
                d: parseInt(document.getElementById(`eq_d_${i}`).value) || 0,
                s: parseInt(document.getElementById(`eq_s_${i}`).value) || 0
            });
        }
        localStorage.setItem('fichaRPG_save', JSON.stringify(estado));
        atualizarUI(estado);
    }

    function atualizarUI(s) {
        const charBase = data.chars[s.char];
        
        // Cálculo Status
        let bH = 0, bD = 0, bS = 0;
        s.equips.forEach(e => { bH += e.h; bD += e.d; bS += e.s; });

        document.getElementById('total-hp').innerText = charBase.hp + bH;
        document.getElementById('total-dmg').innerText = charBase.dmg + bD;
        document.getElementById('total-shd').innerText = bS;

        // Barra XP
        const req = s.lvl * 500;
        document.getElementById('xp-text').innerText = `XP para Nível ${s.lvl + 1}: ${req}`;
        document.getElementById('xp-fill').style.width = Math.min((s.xp / req) * 100, 100) + "%";

        // Render Habilidades
        const container = document.getElementById('skills-container');
        container.innerHTML = `
            <div class="skill-card" style="border-color: #f1e05a">
                <span class="skill-name">PASSIVA: ${charBase.p}</span>
                <div class="skill-desc">${charBase.pd}</div>
            </div>
        `;

        data.skills.forEach(sk => {
            if(s.lvl >= sk.lvl && sk[s.char] !== "") {
                container.innerHTML += `
                    <div class="skill-card">
                        <span class="skill-lvl">Lvl ${sk.lvl}</span>
                        <span class="skill-name">${sk[s.char]}</span>
                        <div class="skill-desc">Habilidade de classe desbloqueada.</div>
                    </div>
                `;
            }
        });
    }

    function carregar() {
        const salvo = localStorage.getItem('fichaRPG_save');
        if(salvo) {
            const s = JSON.parse(salvo);
            document.getElementById('part-name').value = s.part;
            document.getElementById('char-select').value = s.char;
            document.getElementById('lvl').value = s.lvl;
            document.getElementById('xp').value = s.xp;
            document.getElementById('ouro').value = s.ouro;
            s.equips.forEach((e, i) => {
                document.getElementById(`eq_h_${i+1}`).value = e.h || "";
                document.getElementById(`eq_d_${i+1}`).value = e.d || "";
                document.getElementById(`eq_s_${i+1}`).value = e.s || "";
            });
            atualizarUI(s);
        } else {
            salvar();
        }
    }

    function resetar() {
        if(confirm("Deseja apagar todos os dados da ficha?")) {
            localStorage.removeItem('fichaRPG_save');
            location.reload();
        }
    }

    window.onload = carregar;
</script>
</body>
</html>
