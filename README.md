<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Barbearia Michel - Agendamento</title>

<style>
*{
    margin:0;
    padding:0;
    box-sizing:border-box;
    font-family:'Segoe UI',sans-serif;
}

body{
    background:url('https://images.unsplash.com/photo-1585747860715-2ba37e788b70') center/cover no-repeat fixed;
    color:#fff;
}

body::before{
    content:"";
    position:fixed;
    width:100%;
    height:100%;
    background:rgba(0,0,0,0.85);
    z-index:-1;
}

.container{
    max-width:1000px;
    margin:40px auto;
    background:rgba(255,255,255,0.05);
    backdrop-filter:blur(15px);
    padding:30px;
    border-radius:15px;
}

h1{
    text-align:center;
    color:#ff9a00;
    margin-bottom:30px;
}

.section{margin-bottom:25px;}

input,select{
    width:100%;
    padding:12px;
    margin-top:5px;
    border:none;
    border-radius:6px;
    background:rgba(255,255,255,0.1);
    color:#fff;
}

input::placeholder{color:#ccc;}

.calendar{
    display:grid;
    grid-template-columns:repeat(7,1fr);
    gap:10px;
    margin-top:15px;
}

.day{
    padding:10px;
    background:rgba(255,255,255,0.08);
    text-align:center;
    cursor:pointer;
    border-radius:6px;
    transition:0.3s;
}

.day:hover{background:#ff9a00;color:#000;}
.day.selected{background:#ff5a00;color:#000;}

.horarios{
    display:grid;
    grid-template-columns:repeat(5,1fr);
    gap:10px;
    margin-top:15px;
}

.hora{
    padding:10px;
    text-align:center;
    background:rgba(255,255,255,0.08);
    border-radius:6px;
    cursor:pointer;
    transition:0.3s;
}

.hora:hover{background:#ff9a00;color:#000;}
.hora.selected{background:#ff5a00;color:#000;}

.hora.bloqueado{
    background:#333;
    cursor:not-allowed;
    opacity:0.5;
}

button{
    width:100%;
    padding:14px;
    margin-top:20px;
    border:none;
    border-radius:6px;
    background:linear-gradient(90deg,#ff9a00,#ff5a00);
    font-weight:bold;
    cursor:pointer;
}

.resumo{
    display:none;
    margin-top:25px;
    background:rgba(0,0,0,0.7);
    padding:20px;
    border-radius:10px;
}

.whatsapp{
    background:#25d366;
    margin-top:15px;
}
</style>
</head>
<body>

<div class="container">

<h1>Agendamento - Barbearia Michel</h1>

<div class="section">
<label>Nome</label>
<input type="text" id="nome" placeholder="Digite seu nome" required>
</div>

<div class="section">
<label>Telefone</label>
<input type="text" id="telefone" placeholder="(11) 99999-9999" maxlength="15" required>
</div>

<div class="section">
<label>Serviço</label>
<select id="servico" required>
<option value="" data-preco="">Escolha</option>
<option value="Corte Simples" data-preco="40">Corte Simples - R$ 40,00 - 1h</option>
<option value="Corte + Barba" data-preco="75">Corte + Barba - R$ 75,00 - 1h</option>
<option value="Corte + Pigmentação" data-preco="90">Corte + Pigmentação - R$ 90,00 - 1h</option>
<option value="Barba" data-preco="30">Barba - R$ 30,00 - 1h</option>
</select>
</div>

<div class="section">
<label>Selecione o Dia</label>
<div class="calendar" id="calendar"></div>
</div>

<div class="section">
<label>Selecione o Horário (08h às 21h)</label>
<div class="horarios" id="horarios"></div>
</div>

<button onclick="confirmar()">Confirmar Agendamento</button>

<div class="resumo" id="resumo"></div>

</div>

<script>

let dataSelecionada="";
let horaSelecionada="";
const calendar=document.getElementById("calendar");
const hoje=new Date();

// ===== MÁSCARA TELEFONE =====
document.getElementById("telefone").addEventListener("input",function(e){
    let v=e.target.value.replace(/\D/g,"");
    v=v.replace(/^(\d{2})(\d)/g,"($1) $2");
    v=v.replace(/(\d{5})(\d)/,"$1-$2");
    e.target.value=v;
});

// ===== GERAR CALENDÁRIO =====
for(let i=0;i<30;i++){
    let dia=new Date();
    dia.setDate(hoje.getDate()+i);

    let dataISO = dia.getFullYear()+"-"+ 
    String(dia.getMonth()+1).padStart(2,'0')+"-"+ 
    String(dia.getDate()).padStart(2,'0');

    let btn=document.createElement("div");
    btn.className="day";
    btn.innerText=dia.toLocaleDateString("pt-BR",{day:"2-digit",month:"2-digit"});
    btn.dataset.data=dataISO;

    btn.onclick=function(){
        document.querySelectorAll(".day").forEach(d=>d.classList.remove("selected"));
        btn.classList.add("selected");
        dataSelecionada=btn.dataset.data;
        carregarHorarios();
    };

    calendar.appendChild(btn);
}

// ===== HORÁRIOS =====
function carregarHorarios(){
    let container=document.getElementById("horarios");
    container.innerHTML="";
    horaSelecionada="";

    for(let h=8;h<=21;h++){
        let hora=String(h).padStart(2,'0')+":00";
        let btn=document.createElement("div");
        btn.className="hora";
        btn.innerText=hora;

        let chave=dataSelecionada+"_"+hora;

        if(localStorage.getItem(chave)){
            btn.classList.add("bloqueado");
        }else{
            btn.onclick=function(){
                document.querySelectorAll(".hora").forEach(x=>x.classList.remove("selected"));
                btn.classList.add("selected");
                horaSelecionada=hora;
            };
        }

        container.appendChild(btn);
    }
}

// ===== CONFIRMAR =====
function confirmar(){

    let nome=document.getElementById("nome").value.trim();
    let telefone=document.getElementById("telefone").value.trim();
    let servicoSelect=document.getElementById("servico");
    let servico=servicoSelect.value;
    let preco=Number(servicoSelect.options[servicoSelect.selectedIndex].dataset.preco);

    if(!nome||!telefone||!servico||!dataSelecionada||!horaSelecionada){
        alert("Preencha todos os campos!");
        return;
    }

    let chave=dataSelecionada+"_"+horaSelecionada;
    localStorage.setItem(chave,"ocupado");

    let dataFormatada=new Date(dataSelecionada+"T00:00:00").toLocaleDateString("pt-BR");

    let valorFormatado=preco.toLocaleString("pt-BR",{style:"currency",currency:"BRL"});

    document.getElementById("resumo").style.display="block";
    document.getElementById("resumo").innerHTML=`
    <h2>Confirmação</h2>
    <p><strong>Nome:</strong> ${nome}</p>
    <p><strong>Telefone:</strong> ${telefone}</p>
    <p><strong>Serviço:</strong> ${servico}</p>
    <p><strong>Data:</strong> ${dataFormatada}</p>
    <p><strong>Horário:</strong> ${horaSelecionada}</p>
    <p><strong>Valor:</strong> ${valorFormatado}</p>
    <button class="whatsapp" onclick="enviarWhatsApp('${nome}','${servico}','${dataFormatada}','${horaSelecionada}','${valorFormatado}')">
    Confirmar pelo WhatsApp
    </button>
    `;

    carregarHorarios();
}

// ===== WHATSAPP =====
function enviarWhatsApp(nome,servico,data,hora,valor){

let msg=`Olá, gostaria de confirmar meu agendamento na Barbearia Michel:

Nome: ${nome}
Serviço: ${servico}
Data: ${data}
Horário: ${hora}
Valor: ${valor}`;

let link=`https://wa.me/5511957283274?text=${encodeURIComponent(msg)}`;

window.open(link,"_blank");
}

</script>

</body>
</html>
