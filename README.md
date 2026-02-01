import React, { useState, useEffect, useRef } from 'react';
import { 
  BookOpen, 
  Send, 
  Sparkles, 
  Image as ImageIcon,
  Loader2,
  User,
  Trash2,
  AlertTriangle,
  MessageCircle,
  Sun,
  Moon,
  Download,
  XCircle,
  History
} from 'lucide-react';

const apiKey = ""; 

const App = () => {
  const [isDarkMode, setIsDarkMode] = useState(true);
  const [activeTab, setActiveTab] = useState('chat');
  const [query, setQuery] = useState('');
  const [messages, setMessages] = useState([
    { role: 'assistant', text: 'Halo, ada yang bisa saya bantu?' }
  ]);
  const [loading, setLoading] = useState(false);
  const [imagePrompt, setImagePrompt] = useState('');
  const [visualHistory, setVisualHistory] = useState([]);
  const [isGeneratingImg, setIsGeneratingImg] = useState(false);
  const [showDeleteModal, setShowDeleteModal] = useState(false);
  
  const messagesEndRef = useRef(null);

  const scrollToBottom = () => {
    if (messagesEndRef.current) {
      messagesEndRef.current.scrollIntoView({ behavior: "smooth" });
    }
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages, loading]);

  const callGemini = async (userQuery) => {
    setLoading(true);
    try {
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: userQuery }] }],
          systemInstruction: { 
            parts: [{ text: "Kamu adalah asisten akademik skripsi yang santai. JANGAN gunakan sapaan formal seperti Bapak, Ibu, Saudara, atau Selamat Pagi/Siang/Malam. JANGAN gunakan tanda bintang (*) untuk menebalkan teks atau list. Gunakan bahasa Indonesia yang langsung pada intinya. Jika membuat daftar, gunakan angka 1, 2, 3 saja tanpa karakter spesial lainnya. Gunakan nada bicara teman sebaya yang suportif." }] 
          },
          tools: [{ "google_search": {} }]
        })
      });

      const result = await response.json();
      let text = result.candidates?.[0]?.content?.parts?.[0]?.text || "Aduh, koneksi lagi bermasalah. Coba tanya lagi ya.";
      
      text = text.replace(/\*/g, '');

      const sources = result.candidates?.[0]?.groundingMetadata?.groundingAttributions?.map(a => ({
        uri: a.web?.uri,
        title: a.web?.title
      })) || [];

      setMessages(prev => [...prev, { role: 'assistant', text, sources }]);
    } catch (error) {
      setMessages(prev => [...prev, { role: 'assistant', text: "Gagal nyambung ke AI. Cek koneksi internet kamu ya." }]);
    } finally {
      setLoading(false);
    }
  };

  const generateImage = async () => {
    if (!imagePrompt) return;
    setIsGeneratingImg(true);
    try {
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-generate-001:predict?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          instances: { prompt: imagePrompt },
          parameters: { sampleCount: 1 }
        })
      });
      const result = await response.json();
      if (result.predictions && result.predictions[0]) {
        const newImg = {
          id: Date.now(),
          url: `data:image/png;base64,${result.predictions[0].bytesBase64Encoded}`,
          prompt: imagePrompt,
          timestamp: new Date().toLocaleTimeString()
        };
        setVisualHistory(prev => [newImg, ...prev]);
        setImagePrompt('');
      }
    } catch (e) {
      console.error(e);
    } finally {
      setIsGeneratingImg(false);
    }
  };

  const deleteVisual = (id) => {
    setVisualHistory(prev => prev.filter(img => img.id !== id));
  };

  const handleSendMessage = (e) => {
    if (e) e.preventDefault();
    if (!query.trim() || loading) return;
    
    const currentQuery = query.trim();
    setMessages(prev => [...prev, { role: 'user', text: currentQuery }]);
    setQuery('');
    callGemini(currentQuery);
  };

  const clearChat = () => {
    setMessages([{ role: 'assistant', text: 'Halo, ada yang bisa saya bantu?' }]);
    setShowDeleteModal(false);
  };

  // MODIFIED: Menghapus background solid dari themeClass agar gambar terlihat
  const themeClass = isDarkMode ? 'text-slate-200' : 'text-slate-900';
  
  // MODIFIED: Menambahkan transparansi pada komponen UI agar menyatu dengan background
  const headerClass = isDarkMode ? 'bg-[#1e293b]/80 border-slate-800' : 'bg-white/80 border-slate-200';
  const cardClass = isDarkMode ? 'bg-[#1e293b]/90 backdrop-blur-sm border-slate-700' : 'bg-white/90 backdrop-blur-sm border-slate-200';
  const inputClass = isDarkMode ? 'bg-slate-800/80 text-white placeholder:text-slate-500' : 'bg-slate-100/80 text-slate-900 placeholder:text-slate-400';

  return (
    <div className={`min-h-screen flex flex-col relative font-sans transition-colors duration-300 ${themeClass}`}>
      
      {/* BACKGROUND SCENERY & OVERLAY */}
      <div className="fixed inset-0 z-0 pointer-events-none">
        {/* Gambar Pemandangan */}
        <div 
          className="absolute inset-0 bg-cover bg-center transition-all duration-1000 transform scale-[1.02]"
          style={{ 
            backgroundImage: 'url("https://images.unsplash.com/photo-1506744038136-46273834b3fb?auto=format&fit=crop&q=80")',
          }}
        />
        {/* Overlay agar teks tetap terbaca - Gelap di Dark Mode, Putih Susu di Light Mode */}
        <div className={`absolute inset-0 transition-colors duration-500 ${isDarkMode ? 'bg-[#0f172a]/85' : 'bg-slate-50/80'}`} />
      </div>

      {/* HEADER */}
      <header className={`fixed top-0 left-0 right-0 h-16 backdrop-blur-md border-b flex items-center justify-between px-4 md:px-8 z-50 shadow-sm transition-colors duration-300 ${headerClass}`}>
        <div className="flex items-center gap-2">
          <div className="bg-indigo-600 p-1.5 rounded-lg text-white">
            <BookOpen size={20} />
          </div>
          <span className={`font-bold text-lg tracking-tight hidden xs:block ${isDarkMode ? 'text-white' : 'text-slate-900'}`}>SkripsiAI</span>
        </div>
        
        <div className={`flex items-center gap-1 p-1 rounded-xl border transition-colors ${isDarkMode ? 'bg-slate-900/50 border-slate-700' : 'bg-slate-200/50 border-slate-300'}`}>
          <button 
            onClick={() => setActiveTab('chat')}
            className={`flex items-center gap-2 text-[11px] font-bold px-3 py-2 rounded-lg transition-all ${activeTab === 'chat' ? 'bg-indigo-600 text-white shadow-md' : 'text-slate-500 hover:text-indigo-500'}`}
          >
            <MessageCircle size={14} />
            CHAT
          </button>
          <button 
            onClick={() => setActiveTab('visual')}
            className={`flex items-center gap-2 text-[11px] font-bold px-3 py-2 rounded-lg transition-all ${activeTab === 'visual' ? 'bg-indigo-600 text-white shadow-md' : 'text-slate-500 hover:text-indigo-500'}`}
          >
            <ImageIcon size={14} />
            VISUAL
          </button>
        </div>
        
        <div className="flex items-center gap-2">
          <button 
            onClick={() => setIsDarkMode(!isDarkMode)}
            className={`p-2 rounded-lg transition-colors ${isDarkMode ? 'text-yellow-400 hover:bg-slate-800' : 'text-indigo-600 hover:bg-slate-200'}`}
          >
            {isDarkMode ? <Sun size={20} /> : <Moon size={20} />}
          </button>
          <button 
            onClick={() => setShowDeleteModal(true)}
            className="p-2 text-slate-500 hover:text-red-500 transition-colors"
          >
            <Trash2 size={20} />
          </button>
        </div>
      </header>

      {/* MODAL HAPUS CHAT */}
      {showDeleteModal && (
        <div className="fixed inset-0 z-[100] flex items-center justify-center p-4">
          <div className="absolute inset-0 bg-black/60 backdrop-blur-sm" onClick={() => setShowDeleteModal(false)}></div>
          <div className={`relative border rounded-3xl shadow-2xl p-8 w-full max-w-sm animate-in zoom-in duration-200 text-center ${cardClass}`}>
            <div className="w-16 h-16 bg-red-500/10 text-red-500 rounded-full flex items-center justify-center mx-auto mb-4">
              <AlertTriangle size={32} />
            </div>
            <h3 className={`text-xl font-bold ${isDarkMode ? 'text-white' : 'text-slate-900'}`}>Hapus Chat?</h3>
            <p className="text-sm text-slate-500 mt-2 mb-6">Riwayat obrolan bakal hilang permanen.</p>
            <div className="flex gap-3">
              <button onClick={() => setShowDeleteModal(false)} className="flex-1 py-3 font-bold text-slate-400">Batal</button>
              <button onClick={clearChat} className="flex-1 py-3 bg-red-600 text-white rounded-2xl font-bold">Hapus</button>
            </div>
          </div>
        </div>
      )}

      {/* KONTEN */}
      {/* MODIFIED: Added relative and z-10 to make sure content is above background */}
      <main className="flex-1 mt-16 mb-28 w-full overflow-y-auto relative z-10">
        <div className="max-w-3xl mx-auto px-4 py-8">
          {activeTab === 'chat' ? (
            <div className="space-y-6">
              {messages.map((msg, i) => (
                <div key={i} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                  <div className={`flex gap-3 max-w-[95%] md:max-w-[85%] ${msg.role === 'user' ? 'flex-row-reverse' : 'flex-row'}`}>
                    <div className={`w-9 h-9 rounded-xl flex-shrink-0 flex items-center justify-center mt-1 shadow-sm backdrop-blur-sm ${msg.role === 'user' ? 'bg-indigo-600 text-white' : isDarkMode ? 'bg-[#1e293b]/80 text-indigo-400 border border-slate-700' : 'bg-white/80 text-indigo-600 border border-slate-200'}`}>
                      {msg.role === 'user' ? <User size={18} /> : <Sparkles size={18} />}
                    </div>
                    <div className={`px-4 py-3 rounded-2xl shadow-sm border transition-colors backdrop-blur-md ${msg.role === 'user' ? 'bg-indigo-600/90 text-white border-indigo-400' : isDarkMode ? 'bg-[#1e293b]/90 text-slate-200 border-slate-700' : 'bg-white/90 text-slate-800 border-slate-200'}`}>
                      <div className="text-[15px] leading-relaxed space-y-2">
                        {msg.text.split('\n').map((line, idx) => (
                          <p key={idx}>{line}</p>
                        ))}
                      </div>
                      {msg.sources && msg.sources.length > 0 && (
                        <div className={`mt-4 pt-3 border-t flex flex-wrap gap-2 ${isDarkMode ? 'border-slate-700/50' : 'border-slate-100'}`}>
                          {msg.sources.map((s, idx) => (
                            <a key={idx} href={s.uri} target="_blank" rel="noreferrer" className={`text-[10px] px-2 py-1 rounded-md border transition-colors ${isDarkMode ? 'bg-slate-900/50 text-indigo-400 border-slate-700 hover:bg-slate-800' : 'bg-slate-50/50 text-indigo-600 border-slate-200 hover:bg-white'}`}>
                              Sumber {idx + 1}
                            </a>
                          ))}
                        </div>
                      )}
                    </div>
                  </div>
                </div>
              ))}
              {loading && (
                <div className="flex items-center gap-3 text-indigo-500 ml-2 animate-pulse">
                  <Loader2 size={18} className="animate-spin" />
                  <span className="text-sm font-bold tracking-widest uppercase shadow-black/50 drop-shadow-sm">Mencari Jawaban...</span>
                </div>
              )}
              <div ref={messagesEndRef} className="h-4" />
            </div>
          ) : (
            <div className="space-y-10 animate-in fade-in duration-300">
              {/* GENERATOR BOX */}
              <div className={`p-6 md:p-8 rounded-[32px] border shadow-xl space-y-6 transition-colors ${cardClass}`}>
                <div className="space-y-1 text-center">
                  <h2 className={`text-xl font-bold flex items-center justify-center gap-2 ${isDarkMode ? 'text-white' : 'text-slate-900'}`}>
                    <ImageIcon className="text-indigo-500" size={24} />
                    Visualizer
                  </h2>
                  <p className="text-sm text-slate-500">Bikin gambar ilustrasi buat memperjelas materi skripsi kamu.</p>
                </div>
                <textarea 
                  value={imagePrompt}
                  onChange={(e) => setImagePrompt(e.target.value)}
                  className={`w-full h-28 p-5 border rounded-2xl outline-none transition-all resize-none text-[15px] ${isDarkMode ? 'bg-[#0f172a]/60 border-slate-700 text-slate-200 focus:border-indigo-500' : 'bg-slate-50/60 border-slate-200 text-slate-900 focus:border-indigo-500'}`}
                  placeholder="Ketik deskripsi gambar di sini..."
                />
                <button 
                  onClick={generateImage}
                  disabled={isGeneratingImg || !imagePrompt}
                  className="w-full py-4 bg-indigo-600 text-white rounded-2xl font-bold hover:bg-indigo-700 shadow-lg shadow-indigo-500/20 transition-all flex items-center justify-center gap-2"
                >
                  {isGeneratingImg ? <Loader2 className="animate-spin" size={20} /> : <Sparkles size={20} />}
                  {isGeneratingImg ? 'MEMPROSES...' : 'BUAT GAMBAR SEKARANG'}
                </button>
              </div>

              {/* RIWAYAT VISUAL */}
              <div className="space-y-4">
                <div className="flex items-center justify-between px-2">
                  <div className="flex items-center gap-2">
                    <History size={18} className="text-indigo-500" />
                    <h3 className={`font-bold drop-shadow-md ${isDarkMode ? 'text-slate-200' : 'text-slate-700'}`}>Riwayat Visual</h3>
                  </div>
                  {visualHistory.length > 0 && (
                    <button 
                      onClick={() => setVisualHistory([])}
                      className="text-[10px] font-bold text-red-400 hover:text-red-500 uppercase tracking-wider"
                    >
                      Hapus Semua
                    </button>
                  )}
                </div>

                {visualHistory.length === 0 ? (
                  <div className={`p-10 rounded-[32px] border border-dashed text-center space-y-2 backdrop-blur-sm ${isDarkMode ? 'border-slate-700 bg-[#1e293b]/50 text-slate-400' : 'border-slate-300 bg-white/50 text-slate-500'}`}>
                    <ImageIcon size={40} className="mx-auto opacity-20" />
                    <p className="text-sm italic">Belum ada gambar yang dibuat.</p>
                  </div>
                ) : (
                  <div className="grid grid-cols-1 gap-6">
                    {visualHistory.map((img) => (
                      <div key={img.id} className={`group relative p-2 rounded-[32px] border shadow-lg overflow-hidden animate-in slide-in-from-bottom-4 duration-500 ${cardClass}`}>
                        <img src={img.url} alt={img.prompt} className="w-full rounded-[28px]" />
                        <div className="p-4 space-y-3">
                          <p className={`text-xs leading-relaxed ${isDarkMode ? 'text-slate-400' : 'text-slate-500'}`}>
                            <span className="font-bold text-indigo-500">Prompt:</span> {img.prompt}
                          </p>
                          <div className="flex justify-between items-center pt-2 border-t border-slate-700/30">
                            <span className="text-[10px] font-medium text-slate-500">{img.timestamp}</span>
                            <div className="flex items-center gap-3">
                              <button 
                                onClick={() => deleteVisual(img.id)}
                                className="p-1.5 text-slate-500 hover:text-red-500 transition-colors"
                                title="Hapus Gambar"
                              >
                                <XCircle size={18} />
                              </button>
                              <button className="flex items-center gap-1.5 px-3 py-1.5 bg-indigo-600/10 text-indigo-500 rounded-lg text-xs font-bold hover:bg-indigo-600 hover:text-white transition-all">
                                <Download size={14} /> Simpan
                              </button>
                            </div>
                          </div>
                        </div>
                      </div>
                    ))}
                  </div>
                )}
              </div>
            </div>
          )}
        </div>
      </main>

      {/* INPUT BAR */}
      {activeTab === 'chat' && (
        <div className={`fixed bottom-0 left-0 right-0 border-t p-4 md:p-6 z-40 shadow-2xl backdrop-blur-xl transition-colors duration-300 ${isDarkMode ? 'bg-[#0f172a]/80 border-slate-800' : 'bg-white/80 border-slate-200'}`}>
          <div className="max-w-3xl mx-auto flex items-center">
            <form onSubmit={handleSendMessage} className="relative w-full flex items-center">
              <input 
                type="text"
                value={query}
                onChange={(e) => setQuery(e.target.value)}
                placeholder="Tulis pesan kamu di sini..."
                className={`w-full py-4 pl-6 pr-14 border-2 border-transparent rounded-[24px] transition-all outline-none text-[15px] shadow-inner focus:border-indigo-500 ${inputClass}`}
              />
              <button 
                type="submit"
                disabled={loading || !query.trim()}
                className="absolute right-2 p-2.5 bg-indigo-600 text-white rounded-full hover:bg-indigo-700 disabled:opacity-30 transition-all shadow-md"
              >
                <Send size={18} />
              </button>
            </form>
          </div>
        </div>
      )}

    </div>
  );
};

export default App;
