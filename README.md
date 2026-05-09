import React, { useState, useEffect, useRef } from 'react';
import { Sparkles, Share2, Heart, Drum, CheckCircle2, Copy, Trophy, TrendingUp } from 'lucide-react';

// Custom Animations
const style = `
  @keyframes shake {
    0%, 100% { transform: rotate(0deg); }
    25% { transform: rotate(-10deg); }
    75% { transform: rotate(10deg); }
  }
  @keyframes popIn {
    0% { opacity: 0; transform: scale(0.9); }
    100% { opacity: 1; transform: scale(1); }
  }
  @keyframes slideIn {
    0% { transform: scaleX(0); }
    100% { transform: scaleX(1); }
  }
  @keyframes fillBar {
    0% { width: 0%; }
    100% { width: var(--bar-width); }
  }
  .animate-shake { animation: shake 0.2s infinite; }
  .animate-popIn { animation: popIn 0.5s ease-out forwards; }
  .animate-slideIn { animation: slideIn 0.8s ease-out forwards; }
  .bar-fill { animation: fillBar 1s ease-out forwards; }
`;

const ScratchCard = ({ name, onReveal }) => {
  const canvasRef = useRef(null);
  const [isRevealed, setIsRevealed] = useState(false);

  useEffect(() => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    const width = canvas.width;
    const height = canvas.height;

    ctx.fillStyle = '#D1D5DB'; 
    ctx.fillRect(0, 0, width, height);
    ctx.font = 'bold 16px sans-serif';
    ctx.fillStyle = '#9CA3AF';
    ctx.textAlign = 'center';
    ctx.fillText('SCRATCH TO REVEAL', width / 2, height / 2 + 5);

    const handleMove = (e) => {
      if (isRevealed) return;
      const rect = canvas.getBoundingClientRect();
      const x = (e.touches ? e.touches[0].clientX : e.clientX) - rect.left;
      const y = (e.touches ? e.touches[0].clientY : e.clientY) - rect.top;

      ctx.globalCompositeOperation = 'destination-out';
      ctx.beginPath();
      ctx.arc(x, y, 25, 0, Math.PI * 2);
      ctx.fill();
      checkTransparency();
    };

    const checkTransparency = () => {
      const imageData = ctx.getImageData(0, 0, width, height);
      const pixels = imageData.data;
      let transparentPixels = 0;
      for (let i = 3; i < pixels.length; i += 4) {
        if (pixels[i] === 0) transparentPixels++;
      }
      if ((transparentPixels / (pixels.length / 4)) * 100 > 45) {
        setIsRevealed(true);
        onReveal();
      }
    };

    canvas.addEventListener('mousemove', handleMove);
    canvas.addEventListener('touchmove', handleMove);
    return () => {
      canvas.removeEventListener('mousemove', handleMove);
      canvas.removeEventListener('touchmove', handleMove);
    };
  }, [isRevealed, onReveal]);

  return (
    <div className="relative w-full aspect-square bg-white rounded-[2rem] overflow-hidden shadow-sm border-4 border-white">
      <div className="absolute inset-0 flex items-center justify-center bg-amber-50">
        <span className="text-2xl font-bold text-stone-800">{name}</span>
      </div>
      <canvas
        ref={canvasRef}
        width={300}
        height={300}
        className={`absolute inset-0 w-full h-full cursor-crosshair transition-opacity duration-700 ${isRevealed ? 'opacity-0' : 'opacity-100'}`}
      />
    </div>
  );
};

export default function NumeriqueLaunch() {
  const [step, setStep] = useState(0);
  const [revealedCount, setRevealedCount] = useState(0);
  const [isDrumming, setIsDrumming] = useState(false);
  const [selectedLogo, setSelectedLogo] = useState(null);
  const [hasVoted, setHasVoted] = useState(false);

  // LOGO DATA: 4 logos
  const logoOptions = [
    { id: 1, src: '/logo1.jpeg', label: 'Design One' },
    { id: 2, src: '/logo2.jpeg', label: 'Design Two' },
    { id: 3, src: '/logo3.jpeg', label: 'Design Three' },
    { id: 4, src: '/logo4.jpeg', label: 'Design Four' },
  ];

  // Results data - simulated voting results
  const results = logoOptions.map((logo) => ({
    ...logo,
    votes: 15 + (logo.id * 12),
  }));

  const totalVotes = results.reduce((sum, r) => sum + r.votes, 0);
  const resultsWithPercentage = results.map(r => ({
    ...r,
    percentage: Math.round((r.votes / totalVotes) * 100),
  }));

  const winner = resultsWithPercentage.reduce((prev, current) =>
    current.percentage > prev.percentage ? current : prev
  );

  const handleDrumroll = () => {
    setIsDrumming(true);
    setTimeout(() => { setStep(2); setIsDrumming(false); }, 2500);
  };

  return (
    <div className="min-h-screen bg-[#F5F2EB] text-stone-900 p-6 font-sans">
      <style>{style}</style>
      <div className="max-w-4xl mx-auto">
        
        {step === 0 && (
          <div className="text-center space-y-12 animate-popIn py-10">
            <header className="space-y-4">
              <Heart className="w-12 h-12 text-amber-500 mx-auto fill-amber-500" />
              <h1 className="text-4xl md:text-6xl font-bold tracking-tight">Our Hearts Are Full ❤️</h1>
              <p className="text-stone-500 text-lg">The journey begins with three. Scratch to meet the founders.</p>
            </header>
            <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
              {['Ruchika', 'Japnit', 'Sakshi'].map(name => (
                <ScratchCard key={name} name={name} onReveal={() => setRevealedCount(c => c + 1)} />
              ))}
            </div>
            {revealedCount >= 3 && (
              <button onClick={() => setStep(1)} className="px-12 py-5 bg-stone-900 text-white rounded-[2rem] font-bold hover:bg-amber-600 transition-all scale-110">
                Reveal Agency Name
              </button>
            )}
          </div>
        )}

        {step === 1 && (
          <div className="h-[80vh] flex flex-col justify-center items-center space-y-8">
            <h2 className="text-2xl text-stone-400 tracking-[0.3em] uppercase">The reveal...</h2>
            <button onClick={handleDrumroll} className={`p-16 bg-white rounded-full shadow-2xl ${isDrumming ? 'animate-shake' : 'hover:scale-105'}`}>
              <Drum className="w-20 h-20 text-amber-500" />
              <p className="font-bold mt-4">DRUMROLL</p>
            </button>
          </div>
        )}

        {step === 2 && (
          <div className="animate-popIn text-center space-y-12 py-10">
            <div>
              <h1 className="text-7xl md:text-9xl font-black tracking-tighter">NUMERIQUE</h1>
              <p className="text-xl tracking-[0.6em] text-stone-400 font-light">AGENCY</p>
            </div>

            <div className="bg-white p-8 rounded-[2.5rem] shadow-xl border border-stone-100">
              <h3 className="text-2xl font-bold mb-8">Vote for our Visual Identity</h3>
              <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
                {logoOptions.map((logo) => (
                  <button 
                    key={logo.id}
                    onClick={() => !hasVoted && setSelectedLogo(logo.id)}
                    className={`relative aspect-square rounded-2xl border-2 transition-all flex items-center justify-center overflow-hidden
                      ${selectedLogo === logo.id ? 'border-amber-500 bg-amber-50' : 'border-transparent bg-stone-50 hover:bg-white'}`}
                  >
                    {/* LOGO IMAGE PLACEMENT */}
                    <img 
                      src={logo.src} 
                      alt={logo.label} 
                      className="w-full h-full object-contain p-4"
                      onError={(e) => { e.target.style.display = 'none'; e.target.nextSibling.style.display = 'block'; }}
                    />
                    <span className="hidden text-stone-300 italic text-sm">{logo.label}</span>

                    {hasVoted && (
                      <div className="absolute inset-0 bg-stone-900/70 backdrop-blur-sm flex items-center justify-center text-white font-black text-lg animate-popIn">
                        {resultsWithPercentage.find(r => r.id === logo.id)?.percentage}%
                      </div>
                    )}
                  </button>
                ))}
              </div>
              {!hasVoted ? (
                <button 
                  disabled={!selectedLogo}
                  onClick={() => setHasVoted(true)}
                  className="mt-8 px-12 py-4 bg-stone-900 text-white rounded-full disabled:opacity-20 font-bold hover:bg-amber-600 transition-all"
                >
                  Cast Your Vote
                </button>
              ) : (
                <button onClick={() => setStep(3)} className="mt-8 px-12 py-4 bg-amber-500 text-white rounded-full font-bold hover:bg-amber-600 transition-all">
                  See Results
                </button>
              )}
            </div>
          </div>
        )}

        {step === 3 && (
          <div className="animate-popIn text-center space-y-12 py-10">
            <div className="flex items-center justify-center gap-3 mb-8">
              <Trophy className="w-8 h-8 text-amber-500" />
              <h2 className="text-4xl font-bold">Logo Voting Results</h2>
              <Trophy className="w-8 h-8 text-amber-500" />
            </div>

            {/* Winner Announcement */}
            <div className="bg-gradient-to-r from-amber-400 to-amber-600 text-white p-8 rounded-[2rem] shadow-xl">
              <p className="text-sm font-semibold tracking-widest uppercase mb-4 opacity-90">🏆 Winner</p>
              <div className="flex items-center justify-center gap-6 mb-4">
                <img 
                  src={winner.src} 
                  alt={winner.label} 
                  className="w-24 h-24 object-contain"
                  onError={(e) => { e.target.style.display = 'none'; }}
                />
                <div className="text-left">
                  <h3 className="text-3xl font-bold">{winner.label}</h3>
                  <p className="text-xl font-semibold mt-2">{winner.percentage}% of votes</p>
                  <p className="text-sm opacity-90">{winner.votes} total votes</p>
                </div>
              </div>
            </div>

            {/* Results Breakdown */}
            <div className="bg-white p-8 rounded-[2rem] shadow-xl border border-stone-100">
              <h3 className="text-xl font-bold mb-8 text-left">All Results</h3>
              <div className="space-y-6">
                {resultsWithPercentage.map((result, index) => (
                  <div key={result.id} className="space-y-2">
                    <div className="flex items-center justify-between">
                      <div className="flex items-center gap-3 flex-1">
                        <img 
                          src={result.src} 
                          alt={result.label} 
                          className="w-12 h-12 object-contain rounded"
                          onError={(e) => { e.target.style.display = 'none'; }}
                        />
                        <span className="font-semibold text-stone-700">{result.label}</span>
                      </div>
                      <div className="text-right">
                        <span className="font-bold text-lg">{result.percentage}%</span>
                        <span className="text-stone-400 text-sm ml-2">({result.votes} votes)</span>
                      </div>
                    </div>
                    <div className="w-full bg-stone-200 rounded-full h-3 overflow-hidden">
                      <div
                        className="bg-gradient-to-r from-amber-400 to-amber-600 h-full rounded-full bar-fill"
                        style={{
                          '--bar-width': `${result.percentage}%`,
                          animationDelay: `${index * 0.2}s`
                        }}
                      />
                    </div>
                  </div>
                ))}
              </div>

              {/* Stats */}
              <div className="grid grid-cols-3 gap-4 mt-10 pt-8 border-t border-stone-200">
                <div className="text-center">
                  <p className="text-2xl font-bold text-amber-600">{totalVotes}</p>
                  <p className="text-stone-500 text-sm">Total Votes</p>
                </div>
                <div className="text-center">
                  <p className="text-2xl font-bold text-amber-600">4</p>
                  <p className="text-stone-500 text-sm">Designs</p>
                </div>
                <div className="text-center">
                  <p className="text-2xl font-bold text-amber-600">{winner.percentage}%</p>
                  <p className="text-stone-500 text-sm">Winning Vote</p>
                </div>
              </div>
            </div>

            <button onClick={() => setStep(4)} className="px-12 py-4 bg-stone-900 text-white rounded-full font-bold hover:bg-amber-600 transition-all">
              Continue to Launch
            </button>
          </div>
        )}

        {step === 4 && (
          <div className="text-center py-20 animate-popIn space-y-6">
            <Sparkles className="w-24 h-24 text-amber-500 mx-auto" />
            <h1 className="text-5xl font-bold">Welcome to the Era of Numerique.</h1>
            <p className="text-stone-500 max-w-md mx-auto">We are officially live with <span className="font-bold text-amber-600">{winner.label}</span> as our visual identity. Thank you for being part of our reveal.</p>
            <button 
              onClick={() => { navigator.clipboard.writeText(window.location.href); alert("Copied!"); }}
              className="flex items-center gap-2 mx-auto px-6 py-3 border rounded-full hover:bg-white transition-colors font-semibold"
            >
              <Share2 className="w-4 h-4" /> Share Launch Link
            </button>
            <footer className="mt-20 pt-10 border-t flex justify-center gap-10 text-stone-400 font-medium">
              <span>Ruchika</span><span>Japnit</span><span>Sakshi</span>
            </footer>
          </div>
        )}
      </div>
    </div>
  );
}
