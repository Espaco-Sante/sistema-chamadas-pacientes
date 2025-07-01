// URL e chave do Supabase
const supabaseUrl = 'https://lopjgtlxicfgcdibmhdv.supabase.co';
const supabaseKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImxvcGpndGx4aWNmZ2NkaWJtaGR2Iiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTEzNzM5MjksImV4cCI6MjA2Njk0OTkyOX0.1W33xBoOgKQkzp7tr3clW1NpcZlKRS3fO9AJ0MjAWus';
const supabase = supabase.createClient(supabaseUrl, supabaseKey);

// Função para carregar pacientes em espera (área de recepção)
async function carregarPacientes() {
    const { data, error } = await supabase
        .from('chamadas')
        .select('*')
        .eq('status', 'em espera')
        .order('created_at', { ascending: false });

    if (error) {
        console.error('Erro ao carregar pacientes em espera:', error);
        return;
    }

    const listaPacientes = document.getElementById('lista-pacientes');
    listaPacientes.innerHTML = ''; // Limpa a lista antes de adicionar novos pacientes

    data.forEach(paciente => {
        const li = document.createElement('li');
        li.textContent = `${paciente.nome} - Sala: ${paciente.sala}`;
        listaPacientes.appendChild(li);

        const pacienteSelect = document.getElementById('paciente-id');
        const option = document.createElement('option');
        option.value = paciente.id;
        option.textContent = paciente.nome;
        pacienteSelect.appendChild(option);
    });
}

// Função para carregar histórico de chamadas (área de recepção)
async function carregarHistorico() {
    const { data, error } = await supabase
        .from('historico_chamadas')
        .select('*')
        .order('created_at', { ascending: false });

    if (error) {
        console.error('Erro ao carregar histórico de chamadas:', error);
        return;
    }

    const listaHistorico = document.getElementById('historico-lista');
    listaHistorico.innerHTML = '';

    data.forEach(chamada => {
        const li = document.createElement('li');
        li.textContent = `${chamada.nome} - Sala: ${chamada.sala} - Data: ${chamada.data}`;
        listaHistorico.appendChild(li);
    });
}

// Função para chamar paciente (área de recepção)
async function chamarPaciente() {
    const pacienteId = document.getElementById('paciente-id').value;
    const sala = document.getElementById('sala').value;

    if (!pacienteId || !sala) {
        alert('Selecione um paciente e uma sala');
        return;
    }

    const { data, error } = await supabase
        .from('chamadas')
        .update({ status: 'chamado', sala: sala })
        .match({ id: pacienteId });

    if (error) {
        console.error('Erro ao chamar paciente:', error);
        return;
    }

    alert('Paciente chamado com sucesso!');
    carregarPacientes();
    carregarHistorico();
}

// Função para carregar pacientes chamados (área de profissionais)
async function carregarPacientesChamados() {
    const { data, error } = await supabase
        .from('chamadas')
        .select('*')
        .eq('status', 'chamado')
        .order('created_at', { ascending: false });

    if (error) {
        console.error('Erro ao carregar pacientes chamados:', error);
        return;
    }

    const listaPacientesChamados = document.getElementById('lista-pacientes-chamados');
    listaPacientesChamados.innerHTML = '';

    data.forEach(paciente => {
        const li = document.createElement('li');
        li.textContent = `${paciente.nome} - Sala: ${paciente.sala}`;
        listaPacientesChamados.appendChild(li);

        const pacienteSelect = document.getElementById('paciente-profissional-id');
        const option = document.createElement('option');
        option.value = paciente.id;
        option.textContent = paciente.nome;
        pacienteSelect.appendChild(option);
    });
}

// Função para marcar atendimento como concluído (área de profissionais)
async function marcarAtendimentoConcluido() {
    const pacienteId = document.getElementById('paciente-profissional-id').value;

    if (!pacienteId) {
        alert('Selecione um paciente');
        return;
    }

    const { data, error } = await supabase
        .from('chamadas')
        .update({ status: 'atendido' })
        .match({ id: pacienteId });

    if (error) {
        console.error('Erro ao marcar atendimento como concluído:', error);
        return;
    }

    alert('Atendimento concluído com sucesso!');
    carregarPacientesChamados();
}

// Função para adicionar uma nova sala (área de configurações)
async function adicionarSala() {
    const nomeSala = document.getElementById('nova-sala').value;
    
    if (!nomeSala) {
        alert('Digite o nome da sala');
        return;
    }

    const { data, error } = await supabase
        .from('salas')
        .insert([{ nome: nomeSala }]);

    if (error) {
        console.error('Erro ao adicionar sala:', error);
        return;
    }

    alert('Sala adicionada com sucesso!');
    carregarSalas();
}

// Função para carregar as salas existentes (área de configurações)
async function carregarSalas() {
    const { data, error } = await supabase
        .from('salas')
        .select('*');

    if (error) {
        console.error('Erro ao carregar salas:', error);
        return;
    }

    const listaSalas = document.getElementById('lista-salas');
    listaSalas.innerHTML = '';

    data.forEach(sala => {
        const li = document.createElement('li');
        li.textContent = sala.nome;
        listaSalas.appendChild(li);
    });
}

// Função para adicionar um novo profissional (área de configurações)
async function adicionarProfissional() {
    const nomeProfissional = document.getElementById('novo-profissional').value;

    if (!nomeProfissional) {
        alert('Digite o nome do profissional');
        return;
    }

    const { data, error } = await supabase
        .from('profissionais')
        .insert([{ nome: nomeProfissional }]);

    if (error) {
        console.error('Erro ao adicionar profissional:', error);
        return;
    }

    alert('Profissional adicionado com sucesso!');
    carregarProfissionais();
}

// Função para carregar os profissionais existentes (área de configurações)
async function carregarProfissionais() {
    const { data, error } = await supabase
        .from('profissionais')
        .select('*');

    if (error) {
        console.error('Erro ao carregar profissionais:', error);
        return;
    }

    const listaProfissionais = document.getElementById('lista-profissionais');
    listaProfissionais.innerHTML = '';

    data.forEach(profissional => {
        const li = document.createElement('li');
        li.textContent = profissional.nome;
        listaProfissionais.appendChild(li);
    });
}

// Função para salvar configurações de notificações (área de configurações)
async function salvarConfiguracoes() {
    const tipoNotificacao = document.getElementById('tipo-notificacao').value;

    const { data, error } = await supabase
        .from('configuracoes')
        .upsert([{ chave: 'tipo_notificacao', valor: tipoNotificacao }]);

    if (error) {
        console.error('Erro ao salvar configurações:', error);
        return;
    }

    alert('Configurações salvas com sucesso!');
}

// Inicialização ao carregar a página
document.addEventListener('DOMContentLoaded', () => {
    carregarPacientes();
    carregarHistorico();
    carregarPacientesChamados();
    carregarHistoricoAtendimentos();
    carregarSalas();
    carregarProfissionais();

    document.getElementById('chamar-paciente').addEventListener('click', chamarPaciente);
    document.getElementById('atender-paciente').addEventListener('click', marcarAtendimentoConcluido);
    document.getElementById('adicionar-sala').addEventListener('click', adicionarSala);
    document.getElementById('adicionar-profissional').addEventListener('click', adicionarProfissional);
    document.getElementById('salvar-configuracoes').addEventListener('click', salvarConfiguracoes);
});
