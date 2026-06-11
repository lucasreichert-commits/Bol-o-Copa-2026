import React, { useState, useEffect, useMemo } from 'react';
import { Trophy, Calendar, ListPlus, Plus, ChevronDown, ChevronUp, Save, Trash2, Import } from 'lucide-react';

const PARTICIPANTES = [
  "Lucas", "Michele", "Diéssica", "Aline", 
  "Matheus", "Diego", "Roveda", "Ketlyn", "Arthur"
];

// Dados iniciais baseados na prévia do seu arquivo
const JOGOS_INICIAIS = [
  { id: '1', type: 'group', date: '11/06 (Quinta)', group: 'A', teamA: 'México', teamB: 'África do Sul', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '2', type: 'group', date: '11/06 (Quinta)', group: 'A', teamA: 'Coreia do Sul', teamB: 'Tchéquia', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '3', type: 'group', date: '12/06 (Sexta)', group: 'B', teamA: 'Canadá', teamB: 'Bósnia e Herzegovina', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '4', type: 'group', date: '12/06 (Sexta)', group: 'D', teamA: 'Estados Unidos', teamB: 'Paraguai', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '5', type: 'group', date: '13/06 (Sábado)', group: 'B', teamA: 'Catar', teamB: 'Suíça', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '6', type: 'group', date: '13/06 (Sábado)', group: 'C', teamA: 'Brasil', teamB: 'Marrocos', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '7', type: 'group', date: '13/06 (Sábado)', group: 'C', teamA: 'Haiti', teamB: 'Escócia', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '8', type: 'group', date: '13/06 (Sábado)', group: 'D', teamA: 'Austrália', teamB: 'Turquia', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '9', type: 'group', date: '14/06 (Domingo)', group: 'E', teamA: 'Alemanha', teamB: 'Curaçau', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '10', type: 'group', date: '14/06 (Domingo)', group: 'E', teamA: 'Costa do Marfim', teamB: 'Equador', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '11', type: 'group', date: '14/06 (Domingo)', group: 'F', teamA: 'Holanda', teamB: 'Japão', realScoreA: '', realScoreB: '', predictions: {} },
  { id: '12', type: 'group', date: '14/06 (Domingo)', group: 'F', teamA: 'Suécia', teamB: 'Tunísia', realScoreA: '', realScoreB: '', predictions: {} }
];

export default function BolaoApp() {
  const [activeTab, setActiveTab] = useState('jogos');
  const [matches, setMatches] = useState(() => {
    const saved = localStorage.getItem('bolao2026_data');
    if (saved) {
      try { return JSON.parse(saved); } catch (e) { return JOGOS_INICIAIS; }
    }
    return JOGOS_INICIAIS;
  });

  const [csvInput, setCsvInput] = useState('');
  const [expandedMatches, setExpandedMatches] = useState({});

  // Novo jogo manual (Mata-mata)
  const [newMatch, setNewMatch] = useState({ date: '', phase: 'Oitavas', teamA: '', teamB: '' });

  // Salvar no localstorage sempre que matches mudar
  useEffect(() => {
    localStorage.setItem('bolao2026_data', JSON.stringify(matches));
  }, [matches]);

  const toggleMatch = (id) => {
    setExpandedMatches(prev => ({ ...prev, [id]: !prev[id] }));
  };

  const handleRealScoreChange = (id, field, value) => {
    setMatches(matches.map(m => m.id === id ? { ...m, [field]: value } : m));
  };

  const handlePredictionChange = (matchId, participant, field, value) => {
    setMatches(matches.map(m => {
      if (m.id === matchId) {
        const currentPredictions = m.predictions || {};
        const participantPred = currentPredictions[participant] || { scoreA: '', scoreB: '' };
        return {
          ...m,
          predictions: {
            ...currentPredictions,
            [participant]: { ...participantPred, [field]: value }
          }
        };
      }
      return m;
    }));
  };

  const calcularPontos = (realA, realB, predA, predB) => {
    if (realA === '' || realB === '' || predA === undefined || predB === undefined || predA === '' || predB === '') return { pts: 0, cravada: 0, acerto: 0 };
    
    const rA = parseInt(realA);
    const rB = parseInt(realB);
    const pA = parseInt(predA);
    const pB = parseInt(predB);

    if (rA === pA && rB === pB) return { pts: 5, cravada: 1, acerto: 0 }; // Cravou o placar
    
    // Acertou quem ganha ou empate
    const realVencedor = rA > rB ? 'A' : (rA < rB ? 'B' : 'E');
    const predVencedor = pA > pB ? 'A' : (pA < pB ? 'B' : 'E');

    if (realVencedor === predVencedor) return { pts: 3, cravada: 0, acerto: 1 };
    
    return { pts: 0, cravada: 0, acerto: 0 }; // Errou tudo
  };

  const ranking = useMemo(() => {
    const stats = {};
    PARTICIPANTES.forEach(p => stats[p] = { nome: p, pontos: 0, cravadas: 0, acertos: 0 });

    matches.forEach(m => {
      if (m.realScoreA !== '' && m.realScoreB !== '') {
        PARTICIPANTES.forEach(p => {
          const pred = m.predictions?.[p] || {};
          const resultado = calcularPontos(m.realScoreA, m.realScoreB, pred.scoreA, pred.scoreB);
          stats[p].pontos += resultado.pts;
          stats[p].cravadas += resultado.cravada;
          stats[p].acertos += resultado.acerto;
        });
      }
    });

    return Object.values(stats).sort((a, b) => {
      if (b.pontos !== a.pontos) return b.pontos - a.pontos;
      if (b.cravadas !== a.cravadas) return b.cravadas - a.cravadas;
      return b.acertos - a.acertos;
    });
  }, [matches]);

  const importarCSV = () => {
    if (!csvInput.trim()) return;
    const lines = csvInput.trim().split('\n');
    const newMatches = [];
    
    lines.forEach(line => {
      // Ignorar cabeçalho
      if (line.toLowerCase().includes('data,grupo') || !line.includes(',')) return;
      
      const parts = line.split(',');
      if (parts.length >= 3) {
        const dateRaw = parts[0].trim();
        const groupRaw = parts[1].trim();
        const confronto = parts[2].trim();
        
        if (confronto.includes(' x ')) {
          const [teamA, teamB] = confronto.split(' x ');
          newMatches.push({
            id: crypto.randomUUID(),
            type: 'group',
            date: dateRaw,
            group: groupRaw,
            teamA: teamA.trim(),
            teamB: teamB.trim(),
            realScoreA: '',
            realScoreB: '',
            predictions: {}
          });
        }
      }
    });

    if (newMatches.length > 0) {
      setMatches(prev => [...prev, ...newMatches]);
      setCsvInput('');
      alert(`${newMatches.length} jogos importados com sucesso!`);
      setActiveTab('jogos');
    } else {
      alert("Nenhum jogo válido encontrado. Verifique o formato.");
    }
  };

  const addManualMatch = () => {
    if (!newMatch.teamA || !newMatch.teamB) {
      alert("Preencha os nomes dos times!");
      return;
    }
    const match = {
      id: crypto.randomUUID(),
      type: 'knockout',
      date: newMatch.date || 'Data a definir',
      group: newMatch.phase,
      teamA: newMatch.teamA,
      teamB: newMatch.teamB,
      realScoreA: '',
      realScoreB: '',
      predictions: {}
    };
    setMatches([...matches, match]);
    setNewMatch({ date: '', phase: 'Oitavas', teamA: '', teamB: '' });
    setActiveTab('jogos');
  };

  const limparTudo = () => {
    if(window.confirm("Certeza que deseja APAGAR TODOS os dados? Essa ação não pode ser desfeita.")) {
      setMatches(JOGOS_INICIAIS);
      localStorage.removeItem('bolao2026_data');
    }
  };

  const deleteMatch = (id) => {
    if(window.confirm("Deseja excluir este jogo?")) {
      setMatches(matches.filter(m => m.id !== id));
    }
  };

  return (
    <div className="min-h-screen bg-gray-100 text-gray-800 font-sans pb-10">
      {/* Header */}
      <header className="bg-emerald-700 text-white p-4 shadow-md">
        <div className="max-w-4xl mx-auto flex items-center gap-3">
          <Trophy size={32} className="text-yellow-400" />
          <div>
            <h1 className="text-2xl font-bold">Bolão Copa 2026</h1>
            <p className="text-emerald-200 text-sm">Organização de Palpites e Placar</p>
          </div>
        </div>
      </header>

      {/* Navegação */}
      <nav className="bg-white shadow-sm mb-6">
        <div className="max-w-4xl mx-auto flex overflow-x-auto">
          <button onClick={() => setActiveTab('jogos')} className={`flex-1 py-3 px-4 font-semibold text-center border-b-2 flex items-center justify-center gap-2 whitespace-nowrap transition-colors ${activeTab === 'jogos' ? 'border-emerald-600 text-emerald-700' : 'border-transparent text-gray-500 hover:text-gray-700'}`}>
            <Calendar size={18} /> Jogos e Palpites
          </button>
          <button onClick={() => setActiveTab('ranking')} className={`flex-1 py-3 px-4 font-semibold text-center border-b-2 flex items-center justify-center gap-2 whitespace-nowrap transition-colors ${activeTab === 'ranking' ? 'border-emerald-600 text-emerald-700' : 'border-transparent text-gray-500 hover:text-gray-700'}`}>
            <Trophy size={18} /> Ranking
          </button>
          <button onClick={() => setActiveTab('config')} className={`flex-1 py-3 px-4 font-semibold text-center border-b-2 flex items-center justify-center gap-2 whitespace-nowrap transition-colors ${activeTab === 'config' ? 'border-emerald-600 text-emerald-700' : 'border-transparent text-gray-500 hover:text-gray-700'}`}>
            <ListPlus size={18} /> Adicionar/Importar
          </button>
        </div>
      </nav>

      <main className="max-w-4xl mx-auto px-4">
        {/* TAB: JOGOS */}
        {activeTab === 'jogos' && (
          <div className="space-y-6">
            <div className="bg-blue-50 text-blue-800 p-4 rounded-lg text-sm mb-4">
              <p><strong>Regras:</strong> 5 pontos por placar exato (cravada) e 3 pontos por acertar o vencedor ou empate.</p>
            </div>

            {matches.map((match) => (
              <div key={match.id} className="bg-white rounded-xl shadow-md overflow-hidden border border-gray-200">
                <div className="bg-gray-50 p-3 border-b border-gray-200 flex justify-between items-center text-sm text-gray-600">
                  <span>{match.date} • {match.type === 'group' ? `Grupo ${match.group}` : match.group}</span>
                  <button onClick={() => deleteMatch(match.id)} className="text-red-400 hover:text-red-600">
                    <Trash2 size={16} />
                  </button>
                </div>
                
                <div className="p-4">
                  {/* Linha do Jogo Real */}
                  <div className="flex items-center justify-center gap-2 sm:gap-4 mb-4">
                    <div className="flex-1 text-right font-bold text-sm sm:text-base">{match.teamA}</div>
                    <div className="flex items-center gap-2 bg-gray-100 p-2 rounded-lg">
                      <input 
                        type="number" 
                        min="0"
                        className="w-12 h-10 text-center font-bold text-lg border border-gray-300 rounded focus:ring-2 focus:ring-emerald-500 focus:outline-none"
                        value={match.realScoreA}
                        onChange={(e) => handleRealScoreChange(match.id, 'realScoreA', e.target.value)}
                        placeholder="-"
                      />
                      <span className="text-gray-400 font-bold">X</span>
                      <input 
                        type="number" 
                        min="0"
                        className="w-12 h-10 text-center font-bold text-lg border border-gray-300 rounded focus:ring-2 focus:ring-emerald-500 focus:outline-none"
                        value={match.realScoreB}
                        onChange={(e) => handleRealScoreChange(match.id, 'realScoreB', e.target.value)}
                        placeholder="-"
                      />
                    </div>
                    <div className="flex-1 text-left font-bold text-sm sm:text-base">{match.teamB}</div>
                  </div>
                  <div className="text-center text-xs text-gray-400 mb-4 uppercase tracking-widest">Resultado Oficial</div>

                  {/* Accordion de Palpites */}
                  <button 
                    onClick={() => toggleMatch(match.id)}
                    className="w-full py-2 bg-emerald-50 hover:bg-emerald-100 text-emerald-700 font-medium rounded-lg flex items-center justify-center gap-2 transition-colors"
                  >
                    {expandedMatches[match.id] ? <ChevronUp size={18} /> : <ChevronDown size={18} />}
                    Palpites dos Participantes
                  </button>

                  {expandedMatches[match.id] && (
                    <div className="mt-4 grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3">
                      {PARTICIPANTES.map(p => {
                        const pred = match.predictions?.[p] || { scoreA: '', scoreB: '' };
                        return (
                          <div key={p} className="bg-gray-50 border border-gray-200 rounded-lg p-3 flex flex-col gap-2">
                            <span className="font-semibold text-gray-700 text-sm text-center">{p}</span>
                            <div className="flex items-center justify-center gap-2">
                              <input 
                                type="number" min="0" placeholder="-"
                                className="w-10 h-8 text-center border border-gray-300 rounded text-sm"
                                value={pred.scoreA}
                                onChange={(e) => handlePredictionChange(match.id, p, 'scoreA', e.target.value)}
                              />
                              <span className="text-gray-400 text-xs">x</span>
                              <input 
                                type="number" min="0" placeholder="-"
                                className="w-10 h-8 text-center border border-gray-300 rounded text-sm"
                                value={pred.scoreB}
                                onChange={(e) => handlePredictionChange(match.id, p, 'scoreB', e.target.value)}
                              />
                            </div>
                          </div>
                        )
                      })}
                    </div>
                  )}
                </div>
              </div>
            ))}
            
            {matches.length === 0 && (
              <div className="text-center text-gray-500 py-10">
                Nenhum jogo cadastrado. Vá na aba "Adicionar/Importar".
              </div>
            )}
          </div>
        )}

        {/* TAB: RANKING */}
        {activeTab === 'ranking' && (
          <div className="bg-white rounded-xl shadow-md overflow-hidden">
            <div className="p-4 bg-emerald-700 text-white flex justify-between items-center">
              <h2 className="text-lg font-bold flex items-center gap-2"><Trophy size={20} /> Classificação Geral</h2>
            </div>
            <div className="overflow-x-auto">
              <table className="w-full text-left border-collapse">
                <thead>
                  <tr className="bg-gray-100 text-gray-600 text-sm uppercase tracking-wider">
                    <th className="p-4 font-semibold w-16 text-center">Pos</th>
                    <th className="p-4 font-semibold">Participante</th>
                    <th className="p-4 font-semibold text-center">Pts</th>
                    <th className="p-4 font-semibold text-center hidden sm:table-cell">Cravadas (5p)</th>
                    <th className="p-4 font-semibold text-center hidden sm:table-cell">Acertos (3p)</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-gray-200">
                  {ranking.map((user, index) => (
                    <tr key={user.nome} className="hover:bg-gray-50 transition-colors">
                      <td className="p-4 text-center font-bold text-gray-500">
                        {index === 0 ? '🥇' : index === 1 ? '🥈' : index === 2 ? '🥉' : `${index + 1}º`}
                      </td>
                      <td className="p-4 font-bold text-gray-800">{user.nome}</td>
                      <td className="p-4 text-center font-black text-emerald-600 text-lg">{user.pontos}</td>
                      <td className="p-4 text-center text-gray-500 hidden sm:table-cell">{user.cravadas}</td>
                      <td className="p-4 text-center text-gray-500 hidden sm:table-cell">{user.acertos}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        )}

        {/* TAB: CONFIG & IMPORT */}
        {activeTab === 'config' && (
          <div className="space-y-6">
            
            {/* Adicionar Manualmente */}
            <div className="bg-white rounded-xl shadow-md p-6">
              <h2 className="text-lg font-bold mb-4 flex items-center gap-2 text-gray-800">
                <Plus size={20} className="text-emerald-600"/> Adicionar Jogo Manualmente
              </h2>
              <p className="text-sm text-gray-500 mb-4">Ideal para os jogos de Mata-mata onde os times são definidos depois.</p>
              
              <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
                <div>
                  <label className="block text-sm text-gray-600 mb-1">Data/Hora (Opcional)</label>
                  <input type="text" placeholder="Ex: 28/06 16:00" className="w-full border border-gray-300 p-2 rounded focus:ring-2 focus:ring-emerald-500 focus:outline-none" 
                    value={newMatch.date} onChange={e => setNewMatch({...newMatch, date: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm text-gray-600 mb-1">Fase</label>
                  <select className="w-full border border-gray-300 p-2 rounded focus:ring-2 focus:ring-emerald-500 focus:outline-none"
                    value={newMatch.phase} onChange={e => setNewMatch({...newMatch, phase: e.target.value})}>
                    <option value="16 avos">16 avos de final</option>
                    <option value="Oitavas">Oitavas de final</option>
                    <option value="Quartas">Quartas de final</option>
                    <option value="Semifinal">Semifinal</option>
                    <option value="Terceiro Lugar">Disputa 3º Lugar</option>
                    <option value="Final">Final</option>
                  </select>
                </div>
                <div>
                  <label className="block text-sm text-gray-600 mb-1">Time A</label>
                  <input type="text" placeholder="Ex: Brasil" className="w-full border border-gray-300 p-2 rounded focus:ring-2 focus:ring-emerald-500 focus:outline-none"
                    value={newMatch.teamA} onChange={e => setNewMatch({...newMatch, teamA: e.target.value})} />
                </div>
                <div>
                  <label className="block text-sm text-gray-600 mb-1">Time B</label>
                  <input type="text" placeholder="Ex: Argentina" className="w-full border border-gray-300 p-2 rounded focus:ring-2 focus:ring-emerald-500 focus:outline-none"
                    value={newMatch.teamB} onChange={e => setNewMatch({...newMatch, teamB: e.target.value})} />
                </div>
              </div>
              <button onClick={addManualMatch} className="bg-emerald-600 hover:bg-emerald-700 text-white px-4 py-2 rounded font-medium transition-colors w-full sm:w-auto">
                Cadastrar Jogo
              </button>
            </div>

            {/* Importar CSV */}
            <div className="bg-white rounded-xl shadow-md p-6 border-l-4 border-blue-500">
              <h2 className="text-lg font-bold mb-2 flex items-center gap-2 text-gray-800">
                <Import size={20} className="text-blue-500"/> Importar Restante da Planilha (CSV)
              </h2>
              <p className="text-sm text-gray-600 mb-4">
                Abra seu arquivo Excel/CSV no bloco de notas (ou copie as células do Excel), cole o texto abaixo e clique em importar. Ele vai ler no formato: <code>Data, Grupo, Time A x Time B</code>
              </p>
              <textarea 
                className="w-full h-32 border border-gray-300 p-3 rounded font-mono text-sm focus:ring-2 focus:ring-blue-500 focus:outline-none mb-3"
                placeholder="Exemplo:&#10;15/06 (Segunda),H,Espanha x Cabo Verde,&quot;Atlanta, EUA&quot;&#10;15/06 (Segunda),H,Arábia Saudita x Uruguai,&quot;Miami, EUA&quot;"
                value={csvInput}
                onChange={e => setCsvInput(e.target.value)}
              ></textarea>
              <button onClick={importarCSV} className="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded font-medium transition-colors w-full sm:w-auto">
                Processar e Importar Textos
              </button>
            </div>

            {/* Reset */}
            <div className="bg-red-50 rounded-xl shadow-sm border border-red-200 p-6 mt-10">
              <h3 className="text-red-800 font-bold mb-2">Zona de Perigo</h3>
              <p className="text-red-600 text-sm mb-4">Isto apagará todos os placares reais, palpites e jogos criados, voltando para os 12 jogos iniciais originais.</p>
              <button onClick={limparTudo} className="bg-red-600 hover:bg-red-700 text-white px-4 py-2 rounded font-medium transition-colors">
                Apagar todos os dados
              </button>
            </div>
          </div>
        )}
      </main>
    </div>
  );
}
