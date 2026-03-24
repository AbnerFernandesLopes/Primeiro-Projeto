import React, { useMemo, useState } from 'react';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Badge } from '@/components/ui/badge';
import { Progress } from '@/components/ui/progress';
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from '@/components/ui/dialog';
import { AlertTriangle, CheckCircle2, Clock3, Filter, Plus, TrendingUp, Users, Workflow, Search } from 'lucide-react';

const fasesPadrao = ['Kickoff', 'Levantamento', 'Parametrização', 'Treinamento', 'Validação', 'Go-live'];

const statusProjeto = {
  em_dia: 'Em dia',
  risco: 'Em risco',
  atrasado: 'Atrasado',
  travado: 'Travado',
  concluido: 'Concluído',
};

const motivosPadrao = [
  'Cliente não responde',
  'Falta de informações fiscais/contábeis',
  'Pendência de cadastro',
  'Integração/API',
  'Treinamento pendente',
  'Ajuste de sistema',
  'Infraestrutura',
  'Outro',
];

const responsaveisPadrao = ['KM', 'Cliente', 'Terceiro'];

const dadosIniciais = [
  {
    id: 1,
    cliente: 'SPK Confecções',
    patrocinador: 'Mariana',
    responsavelKM: 'Alex',
    responsavelCliente: 'Carlos',
    dataInicio: '2026-03-01',
    dataPrevista: '2026-03-28',
    status: 'risco',
    observacoes: 'Cliente em fase de validação fiscal.',
    fases: [
      { nome: 'Kickoff', status: 'concluido', inicio: '2026-03-01', prevista: '2026-03-02', fim: '2026-03-02', diasParado: 0 },
      { nome: 'Levantamento', status: 'concluido', inicio: '2026-03-03', prevista: '2026-03-06', fim: '2026-03-06', diasParado: 0 },
      { nome: 'Parametrização', status: 'concluido', inicio: '2026-03-07', prevista: '2026-03-14', fim: '2026-03-15', diasParado: 1 },
      { nome: 'Treinamento', status: 'em_andamento', inicio: '2026-03-16', prevista: '2026-03-20', fim: '', diasParado: 2 },
      { nome: 'Validação', status: 'nao_iniciado', inicio: '', prevista: '2026-03-24', fim: '', diasParado: 0 },
      { nome: 'Go-live', status: 'nao_iniciado', inicio: '', prevista: '2026-03-28', fim: '', diasParado: 0 },
    ],
    gargalos: [
      { id: 11, fase: 'Treinamento', motivo: 'Cliente não responde', responsavel: 'Cliente', inicio: '2026-03-18', termino: '', observacao: 'Aguardando confirmação do time financeiro.' },
    ],
  },
  {
    id: 2,
    cliente: 'Têxtil Horizonte',
    patrocinador: 'Rafael',
    responsavelKM: 'Luiz',
    responsavelCliente: 'Bruna',
    dataInicio: '2026-03-05',
    dataPrevista: '2026-04-02',
    status: 'travado',
    observacoes: 'Integração de pedidos ainda pendente.',
    fases: [
      { nome: 'Kickoff', status: 'concluido', inicio: '2026-03-05', prevista: '2026-03-06', fim: '2026-03-06', diasParado: 0 },
      { nome: 'Levantamento', status: 'em_andamento', inicio: '2026-03-07', prevista: '2026-03-12', fim: '', diasParado: 5 },
      { nome: 'Parametrização', status: 'nao_iniciado', inicio: '', prevista: '2026-03-18', fim: '', diasParado: 0 },
      { nome: 'Treinamento', status: 'nao_iniciado', inicio: '', prevista: '2026-03-25', fim: '', diasParado: 0 },
      { nome: 'Validação', status: 'nao_iniciado', inicio: '', prevista: '2026-03-30', fim: '', diasParado: 0 },
      { nome: 'Go-live', status: 'nao_iniciado', inicio: '', prevista: '2026-04-02', fim: '', diasParado: 0 },
    ],
    gargalos: [
      { id: 21, fase: 'Levantamento', motivo: 'Integração/API', responsavel: 'Terceiro', inicio: '2026-03-19', termino: '', observacao: 'Aguardando retorno do fornecedor externo.' },
      { id: 22, fase: 'Levantamento', motivo: 'Falta de informações fiscais/contábeis', responsavel: 'Cliente', inicio: '2026-03-17', termino: '2026-03-18', observacao: 'Cliente enviou parcialmente os dados.' },
    ],
  },
  {
    id: 3,
    cliente: 'Moda Sul',
    patrocinador: 'Fernanda',
    responsavelKM: 'Alex',
    responsavelCliente: 'João',
    dataInicio: '2026-02-10',
    dataPrevista: '2026-03-15',
    status: 'concluido',
    observacoes: 'Projeto finalizado com sucesso.',
    fases: fasesPadrao.map((fase, idx) => ({
      nome: fase,
      status: 'concluido',
      inicio: `2026-02-${String(10 + idx * 3).padStart(2, '0')}`,
      prevista: `2026-02-${String(11 + idx * 3).padStart(2, '0')}`,
      fim: `2026-02-${String(11 + idx * 3).padStart(2, '0')}`,
      diasParado: 0,
    })),
    gargalos: [],
  },
];

function calcularProgresso(fases) {
  const total = fases.length || 1;
  const concluidas = fases.filter((f) => f.status === 'concluido').length;
  return Math.round((concluidas / total) * 100);
}

function faseAtual(fases) {
  return fases.find((f) => f.status === 'em_andamento' || f.status === 'travado')?.nome || fases.find((f) => f.status === 'nao_iniciado')?.nome || 'Concluído';
}

function contarAtivos(gargalos) {
  return gargalos.filter((g) => !g.termino).length;
}

function classeStatus(status) {
  const mapa = {
    em_dia: 'bg-emerald-100 text-emerald-700 border-emerald-200',
    risco: 'bg-amber-100 text-amber-700 border-amber-200',
    atrasado: 'bg-rose-100 text-rose-700 border-rose-200',
    travado: 'bg-red-100 text-red-700 border-red-200',
    concluido: 'bg-slate-100 text-slate-700 border-slate-200',
  };
  return mapa[status] || 'bg-slate-100 text-slate-700 border-slate-200';
}

export default function GestaoImplantacoesKM() {
  const [projetos, setProjetos] = useState(dadosIniciais);
  const [busca, setBusca] = useState('');
  const [filtroStatus, setFiltroStatus] = useState('todos');
  const [selecionado, setSelecionado] = useState(dadosIniciais[0]);
  const [novoProjeto, setNovoProjeto] = useState({
    cliente: '',
    patrocinador: '',
    responsavelKM: '',
    responsavelCliente: '',
    dataInicio: '',
    dataPrevista: '',
    status: 'em_dia',
    observacoes: '',
  });
  const [novoGargalo, setNovoGargalo] = useState({
    fase: 'Kickoff',
    motivo: motivosPadrao[0],
    responsavel: responsaveisPadrao[0],
    inicio: '',
    observacao: '',
  });

  const projetosFiltrados = useMemo(() => {
    return projetos.filter((p) => {
      const okBusca = [p.cliente, p.patrocinador, p.responsavelKM, p.responsavelCliente]
        .join(' ')
        .toLowerCase()
        .includes(busca.toLowerCase());
      const okStatus = filtroStatus === 'todos' ? true : p.status === filtroStatus;
      return okBusca && okStatus;
    });
  }, [projetos, busca, filtroStatus]);

  const metricas = useMemo(() => {
    const total = projetos.length;
    const travados = projetos.filter((p) => p.status === 'travado').length;
    const atrasados = projetos.filter((p) => p.status === 'atrasado').length;
    const concluidos = projetos.filter((p) => p.status === 'concluido').length;
    const ativos = projetos.reduce((acc, p) => acc + contarAtivos(p.gargalos), 0);

    const gargalosPorResponsavel = projetos
      .flatMap((p) => p.gargalos)
      .reduce((acc, g) => {
        acc[g.responsavel] = (acc[g.responsavel] || 0) + 1;
        return acc;
      }, {});

    const topResponsavel = Object.entries(gargalosPorResponsavel).sort((a, b) => b[1] - a[1])[0];

    const gargalosPorMotivo = projetos
      .flatMap((p) => p.gargalos)
      .reduce((acc, g) => {
        acc[g.motivo] = (acc[g.motivo] || 0) + 1;
        return acc;
      }, {});

    const topMotivo = Object.entries(gargalosPorMotivo).sort((a, b) => b[1] - a[1])[0];

    return {
      total,
      travados,
      atrasados,
      concluidos,
      ativos,
      topResponsavel: topResponsavel ? `${topResponsavel[0]} (${topResponsavel[1]})` : 'Sem dados',
      topMotivo: topMotivo ? `${topMotivo[0]} (${topMotivo[1]})` : 'Sem dados',
    };
  }, [projetos]);

  function adicionarProjeto() {
    if (!novoProjeto.cliente || !novoProjeto.dataInicio || !novoProjeto.dataPrevista) return;
    const projeto = {
      id: Date.now(),
      ...novoProjeto,
      fases: fasesPadrao.map((fase) => ({ nome: fase, status: 'nao_iniciado', inicio: '', prevista: '', fim: '', diasParado: 0 })),
      gargalos: [],
    };
    setProjetos((atual) => [projeto, ...atual]);
    setSelecionado(projeto);
    setNovoProjeto({ cliente: '', patrocinador: '', responsavelKM: '', responsavelCliente: '', dataInicio: '', dataPrevista: '', status: 'em_dia', observacoes: '' });
  }

  function adicionarGargalo() {
    if (!selecionado || !novoGargalo.inicio) return;
    const gargalo = { id: Date.now(), termino: '', ...novoGargalo };
    const atualizados = projetos.map((p) => {
      if (p.id !== selecionado.id) return p;
      const novasFases = p.fases.map((f) => (f.nome === novoGargalo.fase ? { ...f, status: 'travado', diasParado: f.diasParado + 1 } : f));
      return { ...p, status: 'travado', gargalos: [gargalo, ...p.gargalos], fases: novasFases };
    });
    setProjetos(atualizados);
    const atualizado = atualizados.find((p) => p.id === selecionado.id);
    setSelecionado(atualizado);
    setNovoGargalo({ fase: 'Kickoff', motivo: motivosPadrao[0], responsavel: responsaveisPadrao[0], inicio: '', observacao: '' });
  }

  function concluirFase(nomeFase) {
    const hoje = new Date().toISOString().slice(0, 10);
    const atualizados = projetos.map((p) => {
      if (p.id !== selecionado.id) return p;
      const novasFases = p.fases.map((f) => {
        if (f.nome === nomeFase) return { ...f, status: 'concluido', fim: hoje, inicio: f.inicio || hoje };
        return f;
      });
      const progresso = calcularProgresso(novasFases);
      const status = progresso === 100 ? 'concluido' : p.status === 'travado' ? 'travado' : 'em_dia';
      return { ...p, fases: novasFases, status };
    });
    setProjetos(atualizados);
    setSelecionado(atualizados.find((p) => p.id === selecionado.id));
  }

  function encerrarGargalo(id) {
    const hoje = new Date().toISOString().slice(0, 10);
    const atualizados = projetos.map((p) => {
      if (p.id !== selecionado.id) return p;
      const novosGargalos = p.gargalos.map((g) => (g.id === id ? { ...g, termino: hoje } : g));
      const aindaAtivos = novosGargalos.some((g) => !g.termino);
      const status = aindaAtivos ? 'travado' : p.status === 'atrasado' ? 'atrasado' : 'em_dia';
      return { ...p, gargalos: novosGargalos, status };
    });
    setProjetos(atualizados);
    setSelecionado(atualizados.find((p) => p.id === selecionado.id));
  }

  return (
    <div className="min-h-screen bg-[#fbfaf3] p-6 text-slate-800">
      <div className="mx-auto max-w-7xl space-y-6">
        <div className="flex flex-col gap-4 rounded-3xl border bg-white p-6 shadow-sm lg:flex-row lg:items-center lg:justify-between">
          <div>
            <div className="text-sm font-medium uppercase tracking-[0.2em] text-[#04ab5c]">KM Sistemas</div>
            <h1 className="mt-2 text-3xl font-semibold">Gestão de Implantações</h1>
            <p className="mt-2 text-sm text-slate-600">Painel para acompanhar andamento, gargalos e métricas dos projetos de implantação.</p>
          </div>
          <Dialog>
            <DialogTrigger asChild>
              <Button className="rounded-2xl bg-[#04ab5c] hover:bg-[#049a53]">
                <Plus className="mr-2 h-4 w-4" /> Novo projeto
              </Button>
            </DialogTrigger>
            <DialogContent className="max-w-2xl rounded-3xl">
              <DialogHeader>
                <DialogTitle>Cadastrar implantação</DialogTitle>
              </DialogHeader>
              <div className="grid gap-3 md:grid-cols-2">
                <Input placeholder="Cliente" value={novoProjeto.cliente} onChange={(e) => setNovoProjeto({ ...novoProjeto, cliente: e.target.value })} />
                <Input placeholder="Patrocinador" value={novoProjeto.patrocinador} onChange={(e) => setNovoProjeto({ ...novoProjeto, patrocinador: e.target.value })} />
                <Input placeholder="Responsável KM" value={novoProjeto.responsavelKM} onChange={(e) => setNovoProjeto({ ...novoProjeto, responsavelKM: e.target.value })} />
                <Input placeholder="Responsável cliente" value={novoProjeto.responsavelCliente} onChange={(e) => setNovoProjeto({ ...novoProjeto, responsavelCliente: e.target.value })} />
                <div>
                  <div className="mb-1 text-xs text-slate-500">Data de início</div>
                  <Input type="date" value={novoProjeto.dataInicio} onChange={(e) => setNovoProjeto({ ...novoProjeto, dataInicio: e.target.value })} />
                </div>
                <div>
                  <div className="mb-1 text-xs text-slate-500">Data prevista</div>
                  <Input type="date" value={novoProjeto.dataPrevista} onChange={(e) => setNovoProjeto({ ...novoProjeto, dataPrevista: e.target.value })} />
                </div>
                <div className="md:col-span-2">
                  <Textarea placeholder="Observações" value={novoProjeto.observacoes} onChange={(e) => setNovoProjeto({ ...novoProjeto, observacoes: e.target.value })} />
                </div>
              </div>
              <div className="flex justify-end">
                <Button className="rounded-2xl bg-[#04ab5c] hover:bg-[#049a53]" onClick={adicionarProjeto}>Salvar implantação</Button>
              </div>
            </DialogContent>
          </Dialog>
        </div>

        <div className="grid gap-4 md:grid-cols-2 xl:grid-cols-6">
          <Card className="rounded-3xl border-0 shadow-sm xl:col-span-1">
            <CardContent className="p-5">
              <div className="flex items-center justify-between">
                <span className="text-sm text-slate-500">Projetos</span>
                <Workflow className="h-4 w-4 text-[#04ab5c]" />
              </div>
              <div className="mt-4 text-3xl font-semibold">{metricas.total}</div>
            </CardContent>
          </Card>
          <Card className="rounded-3xl border-0 shadow-sm xl:col-span-1">
            <CardContent className="p-5">
              <div className="flex items-center justify-between">
                <span className="text-sm text-slate-500">Travados</span>
                <AlertTriangle className="h-4 w-4 text-red-500" />
              </div>
              <div className="mt-4 text-3xl font-semibold">{metricas.travados}</div>
            </CardContent>
          </Card>
          <Card className="rounded-3xl border-0 shadow-sm xl:col-span-1">
            <CardContent className="p-5">
              <div className="flex items-center justify-between">
                <span className="text-sm text-slate-500">Atrasados</span>
                <Clock3 className="h-4 w-4 text-amber-500" />
              </div>
              <div className="mt-4 text-3xl font-semibold">{metricas.atrasados}</div>
            </CardContent>
          </Card>
          <Card className="rounded-3xl border-0 shadow-sm xl:col-span-1">
            <CardContent className="p-5">
              <div className="flex items-center justify-between">
                <span className="text-sm text-slate-500">Concluídos</span>
                <CheckCircle2 className="h-4 w-4 text-emerald-500" />
              </div>
              <div className="mt-4 text-3xl font-semibold">{metricas.concluidos}</div>
            </CardContent>
          </Card>
          <Card className="rounded-3xl border-0 shadow-sm xl:col-span-1">
            <CardContent className="p-5">
              <div className="flex items-center justify-between">
                <span className="text-sm text-slate-500">Gargalos ativos</span>
                <TrendingUp className="h-4 w-4 text-rose-500" />
              </div>
              <div className="mt-4 text-3xl font-semibold">{metricas.ativos}</div>
            </CardContent>
          </Card>
          <Card className="rounded-3xl border-0 shadow-sm xl:col-span-1">
            <CardContent className="p-5">
              <div className="flex items-center justify-between">
                <span className="text-sm text-slate-500">Quem mais trava</span>
                <Users className="h-4 w-4 text-slate-500" />
              </div>
              <div className="mt-4 text-sm font-semibold">{metricas.topResponsavel}</div>
              <div className="mt-2 text-xs text-slate-500">Principal motivo: {metricas.topMotivo}</div>
            </CardContent>
          </Card>
        </div>

        <div className="grid gap-6 xl:grid-cols-[1.1fr,1.4fr]">
          <Card className="rounded-3xl border-0 shadow-sm">
            <CardHeader className="pb-3">
              <div className="flex flex-col gap-3 lg:flex-row lg:items-center lg:justify-between">
                <CardTitle className="text-xl">Carteira de implantações</CardTitle>
                <div className="flex flex-col gap-2 sm:flex-row">
                  <div className="relative">
                    <Search className="pointer-events-none absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-slate-400" />
                    <Input className="rounded-2xl pl-9" placeholder="Buscar cliente ou responsável" value={busca} onChange={(e) => setBusca(e.target.value)} />
                  </div>
                  <div className="flex items-center gap-2">
                    <Filter className="h-4 w-4 text-slate-400" />
                    <Select value={filtroStatus} onValueChange={setFiltroStatus}>
                      <SelectTrigger className="w-[180px] rounded-2xl">
                        <SelectValue placeholder="Status" />
                      </SelectTrigger>
                      <SelectContent>
                        <SelectItem value="todos">Todos</SelectItem>
                        <SelectItem value="em_dia">Em dia</SelectItem>
                        <SelectItem value="risco">Em risco</SelectItem>
                        <SelectItem value="atrasado">Atrasado</SelectItem>
                        <SelectItem value="travado">Travado</SelectItem>
                        <SelectItem value="concluido">Concluído</SelectItem>
                      </SelectContent>
                    </Select>
                  </div>
                </div>
              </div>
            </CardHeader>
            <CardContent className="space-y-3">
              {projetosFiltrados.map((p) => {
                const progresso = calcularProgresso(p.fases);
                const ativo = selecionado?.id === p.id;
                return (
                  <button
                    key={p.id}
                    onClick={() => setSelecionado(p)}
                    className={`w-full rounded-3xl border p-4 text-left transition ${ativo ? 'border-[#04ab5c] bg-[#f4fff8] shadow-sm' : 'border-slate-200 bg-white hover:border-slate-300'}`}
                  >
                    <div className="flex flex-col gap-3 md:flex-row md:items-start md:justify-between">
                      <div>
                        <div className="text-lg font-semibold">{p.cliente}</div>
                        <div className="mt-1 text-sm text-slate-500">KM: {p.responsavelKM || '-'} • Cliente: {p.responsavelCliente || '-'}</div>
                        <div className="mt-2 text-sm text-slate-600">Fase atual: <span className="font-medium">{faseAtual(p.fases)}</span></div>
                      </div>
                      <Badge className={`rounded-full border ${classeStatus(p.status)}`}>{statusProjeto[p.status]}</Badge>
                    </div>
                    <div className="mt-4 grid gap-3 md:grid-cols-3">
                      <div>
                        <div className="mb-1 text-xs text-slate-500">Progresso</div>
                        <Progress value={progresso} className="h-2" />
                        <div className="mt-1 text-xs text-slate-500">{progresso}% concluído</div>
                      </div>
                      <div>
                        <div className="text-xs text-slate-500">Início</div>
                        <div className="text-sm font-medium">{p.dataInicio || '-'}</div>
                      </div>
                      <div>
                        <div className="text-xs text-slate-500">Prevista</div>
                        <div className="text-sm font-medium">{p.dataPrevista || '-'}</div>
                      </div>
                    </div>
                  </button>
                );
              })}
            </CardContent>
          </Card>

          <Card className="rounded-3xl border-0 shadow-sm">
            <CardHeader>
              <div className="flex flex-col gap-3 lg:flex-row lg:items-center lg:justify-between">
                <div>
                  <CardTitle className="text-xl">{selecionado?.cliente || 'Selecione um projeto'}</CardTitle>
                  <p className="mt-1 text-sm text-slate-500">Acompanhe fases, gargalos e observações do projeto.</p>
                </div>
                {selecionado && <Badge className={`rounded-full border ${classeStatus(selecionado.status)}`}>{statusProjeto[selecionado.status]}</Badge>}
              </div>
            </CardHeader>
            <CardContent>
              {selecionado ? (
                <Tabs defaultValue="fases" className="space-y-4">
                  <TabsList className="grid w-full grid-cols-3 rounded-2xl bg-slate-100">
                    <TabsTrigger value="fases" className="rounded-2xl">Fases</TabsTrigger>
                    <TabsTrigger value="gargalos" className="rounded-2xl">Gargalos</TabsTrigger>
                    <TabsTrigger value="resumo" className="rounded-2xl">Resumo</TabsTrigger>
                  </TabsList>

                  <TabsContent value="fases" className="space-y-3">
                    {selecionado.fases.map((f) => (
                      <div key={f.nome} className="rounded-3xl border border-slate-200 p-4">
                        <div className="flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
                          <div>
                            <div className="font-semibold">{f.nome}</div>
                            <div className="mt-1 text-sm text-slate-500">Início: {f.inicio || '-'} • Prevista: {f.prevista || '-'} • Fim: {f.fim || '-'}</div>
                          </div>
                          <div className="flex items-center gap-2">
                            <Badge variant="secondary" className="rounded-full">{f.status.replace('_', ' ')}</Badge>
                            {f.status !== 'concluido' && (
                              <Button variant="outline" className="rounded-2xl" onClick={() => concluirFase(f.nome)}>Concluir fase</Button>
                            )}
                          </div>
                        </div>
                        {f.diasParado > 0 && <div className="mt-3 text-sm text-rose-600">Dias parados registrados: {f.diasParado}</div>}
                      </div>
                    ))}
                  </TabsContent>

                  <TabsContent value="gargalos" className="space-y-4">
                    <div className="rounded-3xl border border-dashed border-slate-300 p-4">
                      <div className="mb-3 text-sm font-medium">Registrar novo gargalo</div>
                      <div className="grid gap-3 md:grid-cols-2">
                        <Select value={novoGargalo.fase} onValueChange={(v) => setNovoGargalo({ ...novoGargalo, fase: v })}>
                          <SelectTrigger className="rounded-2xl"><SelectValue /></SelectTrigger>
                          <SelectContent>{selecionado.fases.map((f) => <SelectItem key={f.nome} value={f.nome}>{f.nome}</SelectItem>)}</SelectContent>
                        </Select>
                        <Select value={novoGargalo.motivo} onValueChange={(v) => setNovoGargalo({ ...novoGargalo, motivo: v })}>
                          <SelectTrigger className="rounded-2xl"><SelectValue /></SelectTrigger>
                          <SelectContent>{motivosPadrao.map((m) => <SelectItem key={m} value={m}>{m}</SelectItem>)}</SelectContent>
                        </Select>
                        <Select value={novoGargalo.responsavel} onValueChange={(v) => setNovoGargalo({ ...novoGargalo, responsavel: v })}>
                          <SelectTrigger className="rounded-2xl"><SelectValue /></SelectTrigger>
                          <SelectContent>{responsaveisPadrao.map((r) => <SelectItem key={r} value={r}>{r}</SelectItem>)}</SelectContent>
                        </Select>
                        <Input type="date" className="rounded-2xl" value={novoGargalo.inicio} onChange={(e) => setNovoGargalo({ ...novoGargalo, inicio: e.target.value })} />
                        <div className="md:col-span-2">
                          <Textarea placeholder="Observação do gargalo" value={novoGargalo.observacao} onChange={(e) => setNovoGargalo({ ...novoGargalo, observacao: e.target.value })} />
                        </div>
                      </div>
                      <div className="mt-3 flex justify-end">
                        <Button className="rounded-2xl bg-[#04ab5c] hover:bg-[#049a53]" onClick={adicionarGargalo}>Registrar gargalo</Button>
                      </div>
                    </div>

                    <div className="space-y-3">
                      {selecionado.gargalos.length === 0 && (
                        <div className="rounded-3xl border border-slate-200 p-6 text-center text-sm text-slate-500">Nenhum gargalo registrado neste projeto.</div>
                      )}
                      {selecionado.gargalos.map((g) => (
                        <div key={g.id} className="rounded-3xl border border-slate-200 p-4">
                          <div className="flex flex-col gap-3 md:flex-row md:items-start md:justify-between">
                            <div>
                              <div className="font-semibold">{g.motivo}</div>
                              <div className="mt-1 text-sm text-slate-500">Fase: {g.fase} • Responsável: {g.responsavel}</div>
                              <div className="mt-1 text-sm text-slate-500">Início: {g.inicio} • Término: {g.termino || 'Em aberto'}</div>
                              {g.observacao && <div className="mt-2 text-sm text-slate-700">{g.observacao}</div>}
                            </div>
                            {!g.termino && <Button variant="outline" className="rounded-2xl" onClick={() => encerrarGargalo(g.id)}>Encerrar</Button>}
                          </div>
                        </div>
                      ))}
                    </div>
                  </TabsContent>

                  <TabsContent value="resumo" className="space-y-4">
                    <div className="grid gap-4 md:grid-cols-2">
                      <div className="rounded-3xl border border-slate-200 p-4">
                        <div className="text-sm text-slate-500">Patrocinador</div>
                        <div className="mt-1 font-semibold">{selecionado.patrocinador || '-'}</div>
                      </div>
                      <div className="rounded-3xl border border-slate-200 p-4">
                        <div className="text-sm text-slate-500">Responsáveis</div>
                        <div className="mt-1 font-semibold">KM: {selecionado.responsavelKM || '-'} • Cliente: {selecionado.responsavelCliente || '-'}</div>
                      </div>
                      <div className="rounded-3xl border border-slate-200 p-4">
                        <div className="text-sm text-slate-500">Progresso geral</div>
                        <div className="mt-2"><Progress value={calcularProgresso(selecionado.fases)} className="h-2" /></div>
                        <div className="mt-2 text-sm font-medium">{calcularProgresso(selecionado.fases)}%</div>
                      </div>
                      <div className="rounded-3xl border border-slate-200 p-4">
                        <div className="text-sm text-slate-500">Gargalos ativos</div>
                        <div className="mt-1 text-2xl font-semibold">{contarAtivos(selecionado.gargalos)}</div>
                      </div>
                    </div>
                    <div className="rounded-3xl border border-slate-200 p-4">
                      <div className="text-sm text-slate-500">Observações</div>
                      <div className="mt-2 text-sm text-slate-700">{selecionado.observacoes || 'Sem observações registradas.'}</div>
                    </div>
                  </TabsContent>
                </Tabs>
              ) : (
                <div className="rounded-3xl border border-dashed border-slate-300 p-10 text-center text-slate-500">Selecione um projeto para visualizar os detalhes.</div>
              )}
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}
