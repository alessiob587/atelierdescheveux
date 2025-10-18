[index.html](https://github.com/user-attachments/files/22986417/index.html)
<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gestionale Salone - L'Atelier des Cheveux</title>
<link rel="manifest" href="manifest.json">
<style>
    body { font-family: Arial, sans-serif; margin: 0; padding: 0; background: #fff; color: #000; }
    header { background: #000; color: #fff; padding: 1rem; text-align: center; }
    nav { display: flex; justify-content: space-around; background: #000; padding: 0.5rem 0; }
    nav button { padding: 0.5rem 1rem; font-size: 1rem; cursor: pointer; border: none; border-radius: 5px; background: #fff; color: #000; }
    section { padding: 1rem; }
    input, select, textarea, button { width: 100%; padding: 0.5rem; margin: 0.3rem 0; font-size: 1rem; }
    .hidden { display: none; }
    .calendar-header { display: grid; grid-template-columns: repeat(7, 1fr); text-align: center; margin-bottom: 5px; font-size: 1.1rem; }
    .calendar { display: grid; grid-template-columns: repeat(7, 1fr); gap: 10px; }
    .calendar-cell { background: #f0f0f0; border-radius: 5px; padding: 0.5rem; min-height: 350px; }
    .calendar-date { font-weight: bold; margin-bottom: 0.5rem; color: #000; font-size: 1.2rem; }
    .time-slot { font-size: 1.2rem; margin: 10px 0; border-bottom: 1px solid #ccc; padding: 8px 0; }
    .appointment-item { background: #000; color: #fff; margin: 4px 0; padding: 5px 6px; border-radius: 3px; font-size: 1.1rem; }
</style>
</head>
<body>
<header>
    <h1>L'Atelier des Cheveux - Gestionale</h1>
</header>
<nav>
    <button id="btnClients">Clienti</button>
    <button id="btnCalendar">Appuntamenti</button>
    <button id="btnServices">Servizi</button>
</nav>

<section id="clients">
    <h2>Gestione Clienti</h2>
    <input type="text" id="clientName" placeholder="Nome Cliente">
    <input type="text" id="clientPhone" placeholder="Telefono">
    <textarea id="clientNotes" placeholder="Note..."></textarea>
    <button id="addClientBtn">Aggiungi Cliente</button>
    <h3>Lista Clienti</h3>
    <div id="clientList"></div>
</section>

<section id="calendar" class="hidden">
    <h2>Appuntamenti - Planning Settimanale</h2>
    <label>Seleziona giorno:</label>
    <input type="date" id="appointmentDate">
    <label>Cliente:</label>
    <select id="appointmentClient"></select>
    <label>Servizi:</label>
    <div id="appointmentServiceContainer"></div>
    <button id="addAppointmentBtn">Aggiungi Appuntamento</button>

    <div class="calendar-header">
        <div>Lunedì</div>
        <div>Martedì</div>
        <div>Mercoledì</div>
        <div>Giovedì</div>
        <div>Venerdì</div>
        <div>Sabato</div>
        <div>Domenica</div>
    </div>
    <div class="calendar" id="weeklyCalendar"></div>
</section>

<section id="services" class="hidden">
    <h2>Servizi</h2>
    <h3>Servizi Stilistici</h3>
    <ul id="stylistServices"></ul>
    <h3>Servizi Tecnici</h3>
    <ul id="technicalServices"></ul>
    <h3>Trattamenti e Massaggi</h3>
    <ul id="treatmentServices"></ul>
</section>

<script>
let clients = [];
let appointments = [];
const stylistServices = [ /*...*/ ];
const technicalServices = [ /*...*/ ];
const treatmentServices = [ /*...*/ ];
const timeSlots = ['10:00','10:30','11:00','11:30','12:00','12:30','13:00','13:30','14:00','14:30','15:00','15:30','16:00','16:30','17:00','17:30','18:00','18:30','19:00','19:30'];
const holidays = ['2025-01-01','2025-04-25','2025-05-01','2025-06-02','2025-08-15','2025-11-01','2025-12-25','2025-12-26'];

function showSection(sectionId) {
    document.querySelectorAll('section').forEach(sec => sec.classList.add('hidden'));
    document.getElementById(sectionId).classList.remove('hidden');
    if(sectionId === 'services') renderServices();
    if(sectionId === 'calendar') {
        renderClientsDropdown();
        renderServicesCheckboxes();
        renderWeeklyCalendar();
    }
}

function renderServicesCheckboxes() {
    const container = document.getElementById('appointmentServiceContainer');
    container.innerHTML = '';
    [...stylistServices, ...technicalServices, ...treatmentServices].forEach((s, idx) => {
        const label = document.createElement('label');
        label.style.display = 'block';
        const checkbox = document.createElement('input');
        checkbox.type = 'checkbox';
        checkbox.value = s.name;
        label.appendChild(checkbox);
        label.appendChild(document.createTextNode(' ' + s.name));
        container.appendChild(label);
    });
}

function submitAppointment() {
    const date = document.getElementById('appointmentDate').value;
    const clientName = document.getElementById('appointmentClient').value;
    const selectedServices = Array.from(document.querySelectorAll('#appointmentServiceContainer input[type=checkbox]:checked')).map(cb => cb.value);
    if(date && clientName && selectedServices.length) addAppointment(date, clientName, selectedServices);
    else alert('Compila tutti i campi');
}

function addAppointment(date, clientName, services) {
    appointments.push({date, client: {name: clientName}, services});
    renderWeeklyCalendar();
}

function renderWeeklyCalendar() {
    const calendarDiv = document.getElementById('weeklyCalendar');
    calendarDiv.innerHTML = '';
    const today = new Date();
    const start = today.getDate() - today.getDay() + 1;
    for(let i=0; i<7; i++) {
        const day = new Date(today.setDate(start + i));
        const dayStr = day.toISOString().split('T')[0];
        const cell = document.createElement('div');
        cell.className = 'calendar-cell';
        const isHoliday = holidays.includes(dayStr);
        let cellContent = `<div class='calendar-date'>${dayStr}${isHoliday ? ' 🎉' : ''}</div>`;
        timeSlots.forEach(time => cellContent += `<div class='time-slot'>${time}</div>`);
        cell.innerHTML = cellContent;
        appointments.filter(a => a.date === dayStr).forEach(a => {
            const item = document.createElement('div');
            item.className = 'appointment-item';
            item.textContent = `${a.client.name} - ${a.services.join(', ')}`;
            cell.appendChild(item);
        });
        calendarDiv.appendChild(cell);
    }
}

// Event listeners
document.getElementById('btnClients').addEventListener('click', () => showSection('clients'));
document.getElementById('btnCalendar').addEventListener('click', () => showSection('calendar'));
document.getElementById('btnServices').addEventListener('click', () => showSection('services'));
document.getElementById('addAppointmentBtn').addEventListener('click', submitAppointment);

// PWA: register service worker
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('service-worker.js').then(() => console.log('Service Worker registrato'));
}
</script>
</body>
</html>
