# Blackjack-mini-app
Kral Şengül, [27.11.2025 10:16]
import React, { useState, useEffect, useCallback, useMemo } from 'react';
// Base Mini App SDK import edildi (Projeye npm ile yüklenmesi gerekir)
// BaseApp ortamında SDK global olarak sağlanmazsa bu import hata verecektir, 
// ancak Base Quickstart şablonu bu bağımlılığı npm ile otomatik kurar.
import { SDK } from '@farcaster/mini-app-sdk'; 

// Kart Tanımları
const RANKS = ['A', '2', '3', '4', '5', '6', '7', '8', '9', '10', 'J', 'Q', 'K'];
const SUITS = ['♠️', '♥️', '♦️', '♣️'];

// ===========================================
// OYUN MANTIĞI YARDIMCI FONKSİYONLARI
// ===========================================

// Kartın numerik değerini hesaplar (Ace=11 veya 1)
const getCardValue = (card) => {
    const rank = card.rank;
    if (['K', 'Q', 'J'].includes(rank)) return 10;
    if (rank === 'A') return 11;
    return parseInt(rank, 10);
};

// Elin toplam değerini hesaplar (Ace 11/1 yönetimi dahil)
const calculateHandValue = (hand) => {
    let value = hand.reduce((sum, card) => sum + getCardValue(card), 0);
    let aceCount = hand.filter(card => card.rank === 'A').length;

    // As (Ace) yönetimi: Toplam 21'i geçerse 1 olarak say
    while (value > 21 && aceCount > 0) {
        value -= 10; // 11'i 1'e çevir
        aceCount--;
    }
    return value;
};

// Yeni kart destesi oluşturur ve karıştırır
const createAndShuffleDeck = () => {
    let deck = [];
    for (const suit of SUITS) {
        for (const rank of RANKS) {
            deck.push({ rank, suit });
        }
    }
    // Fisher-Yates shuffle algoritması
    for (let i = deck.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [deck[i], deck[j]] = [deck[j], deck[i]];
    }
    return deck;
};

// ===========================================
// REACT BİLEŞENLERİ
// ===========================================

// Kart Bileşeni
const Card = ({ card, hidden = false }) => {
    const colorClass = card?.suit === '♥️' || card?.suit === '♦️' ? 'text-red-700' : 'text-gray-900';

    if (hidden) {
        return (
            <div className="w-24 h-36 sm:w-28 sm:h-40 bg-gray-700 rounded-lg shadow-lg flex flex-col justify-center items-center p-3 m-1 border-2 border-dashed border-gray-400">
                <span className="text-5xl text-gray-400">?</span>
            </div>
        );
    }
    
    // Kart açıkken
    return (
        <div className={w-24 h-36 sm:w-28 sm:h-40 bg-white rounded-lg shadow-lg p-2 m-1 border-2 border-gray-900 relative ${colorClass} text-gray-900}>
            {/* RÜTBE (Sol Üst Köşe) */}
            <div className="absolute top-2 left-2 text-3xl font-extrabold" style={{lineHeight: '1'}}>
                {card.rank}
            </div>
            
            {/* SEMBOL (Merkez) */}
            <div className="absolute inset-0 flex justify-center items-center">
                <span className={text-7xl ${colorClass}}>{card.suit}</span>
            </div>

            {/* Ters RÜTBE (Sağ Alt Köşe) */}
            <div className="absolute bottom-2 right-2 text-3xl font-extrabold rotate-180" style={{lineHeight: '1'}}>
                {card.rank}
            </div>
        </div>
    );
};

// Ana Uygulama Bileşeni
const App = () => {
    const [chips, setChips] = useState(1000); 
    const [deck, setDeck] = useState(createAndShuffleDeck());
    const [playerHand, setPlayerHand] = useState([]);
    const [dealerHand, setDealerHand] = useState([]);
    const [bet, setBet] = useState(50);
    const [currentBet, setCurrentBet] = useState(0);
    const [gameState, setGameState] = useState('BETTING'); 
    const [message, setMessage] = useState('Bahis yapın ve oynamaya başlayın!');
    const [winner, setWinner] = useState(null); 
    
    // VİDEODAKİ GİBİ: Base App'e hazır olduğunu bildir
    useEffect(() => {
        // SDK'nın varlığını kontrol et

Kral Şengül, [27.11.2025 10:16]
if (typeof SDK !== 'undefined' && SDK.Actions && SDK.Actions.ready) {
            // Hiçbir yükleme mantığı olmadığı için doğrudan hazır (ready) çağırılabilir.
            SDK.Actions.ready();
            console.log("Base Mini App SDK: Uygulama hazır (ready) çağrısı yapıldı.");
        } else {
            console.warn("Base Mini App SDK bulunamadı veya düzgün yüklenmedi.");
        }
    }, []); // Sadece bileşen ilk yüklendiğinde çalışır (Videodaki tavsiye)
    // ----------------------------------------------------

    // El Değerlerini Hesaplama
    const playerValue = useMemo(() => calculateHandValue(playerHand), [playerHand]);
    const dealerValue = useMemo(() => calculateHandValue(dealerHand), [dealerHand]);
    const dealerVisibleValue = useMemo(() => dealerHand.length > 0 ? getCardValue(dealerHand[0]) : 0, [dealerHand]);

    // Kart çekme fonksiyonu
    const drawCard = useCallback(() => {
        if (deck.length === 0) {
            setMessage('Deste bitiyor! Deste karıştırılıyor...');
            setDeck(createAndShuffleDeck());
        }
        const card = deck[0];
        setDeck(prevDeck => prevDeck.slice(1));
        return card;
    }, [deck]);

    // ===========================================
    // OYUN AKIŞI FONKSİYONLARI
    // ===========================================

    // Bahis Yap
    const placeBet = (amount) => {
        if (gameState !== 'BETTING') return;
        const newBet = Math.min(chips, Math.max(0, amount));
        setBet(newBet);
    };

    // Oyunu Başlat: İlk Kartları Dağıt
    const dealInitialCards = () => {
        if (currentBet === 0 || chips < currentBet) {
            setMessage('Lütfen geçerli bir bahis yapın.');
            return;
        }

        let newDeck = [...deck];
        let pHand = [];
        let dHand = [];

        // Hızlı kart dağıtımı için:
        for (let i = 0; i < 2; i++) {
            pHand.push(newDeck.shift());
            dHand.push(newDeck.shift());
        }

        setDeck(newDeck);
        setPlayerHand(pHand);
        setDealerHand(dHand);
        setChips(prev => prev - currentBet);
        setGameState('PLAYER_TURN');
        setMessage('Sıra sizde: Vur (Hit) veya Dur (Stand)?');

        // Blackjack kontrolü
        const pValue = calculateHandValue(pHand);
        if (pValue === 21) {
            const dValue = calculateHandValue(dHand);
            if (dValue === 21) {
                // Dealer Blackjack! Kontrolü checkWinner'a bırak
            } else {
                setWinner('BLACKJACK');
                setMessage('BLACKJACK! Kazandınız!');
                setGameState('RESULT');
            }
        }
    };

    // Kart Çek (Hit)
    const handleHit = () => {
        if (gameState !== 'PLAYER_TURN') return;

        const newCard = drawCard();
        const newHand = [...playerHand, newCard];
        setPlayerHand(newHand);

        const newValue = calculateHandValue(newHand);
        if (newValue > 21) {
            // Bust
            setWinner('DEALER');
            setMessage(Aştınız (${newValue})! Krupiye kazandı.);
            setGameState('RESULT');
        }
    };

    // Dur (Stand)
    const handleStand = () => {
        if (gameState !== 'PLAYER_TURN') return;
        setMessage('Krupiye oynuyor...');
        setGameState('DEALER_TURN');
    };

    // İkiye Katla (Double Down)
    const handleDoubleDown = () => {
        if (gameState !== 'PLAYER_TURN' || playerHand.length > 2) return;
        
        // Bahis kontrolü
        if (chips < currentBet) {
            setMessage('İkiye katlamak için yeterli paranız yok!');
            return;
        }

        // İkinci bahsi koy ve bir kart çek
        setChips(prev => prev - currentBet);
        setCurrentBet(prev => prev * 2);

        const newCard = drawCard();
        const newHand = [...playerHand, newCard];
        setPlayerHand(newHand);

Kral Şengül, [27.11.2025 10:16]
const newValue = calculateHandValue(newHand);
        if (newValue > 21) {
             // Bust
            setWinner('DEALER');
            setMessage(Aştınız (${newValue})! İkiye katladınız ve kaybettiniz.);
            setGameState('RESULT');
        } else {
            // Stand yap ve krupiye oynasın
            setMessage('İkiye katlandı. Krupiye oynuyor...');
            setGameState('DEALER_TURN');
        }
    };

    // Krupiye Oynar
    const dealerPlay = useCallback(() => {
        let currentDeck = [...deck];
        let dHand = [...dealerHand];
        let dValue = calculateHandValue(dHand);
        let updatedChips = chips;

        // Krupiye 17'ye ulaşana kadar çeker
        while (dValue < 17) {
            const newCard = currentDeck.shift();
            if (!newCard) break; // Deste biterse dur
            dHand.push(newCard);
            dValue = calculateHandValue(dHand);
        }

        setDeck(currentDeck);
        setDealerHand(dHand);
        setGameState('RESULT');
        checkWinner(playerValue, dValue, updatedChips);

    }, [deck, dealerHand, playerValue, chips]);

    // Kazananı Belirle ve Çipleri Güncelle
    const checkWinner = useCallback((pVal, dVal) => {
        let finalWinner = '';
        let chipChange = 0;

        if (pVal > 21) {
            finalWinner = 'DEALER'; // Zaten handled
        } else if (dVal > 21) {
            finalWinner = 'PLAYER';
            chipChange = currentBet * 2;
        } else if (pVal > dVal) {
            finalWinner = 'PLAYER';
            chipChange = currentBet * 2;
        } else if (pVal < dVal) {
            finalWinner = 'DEALER';
        } else {
            finalWinner = 'PUSH';
            chipChange = currentBet;
        }

        setWinner(finalWinner);
        
        // Blackjack 1.5x öder 
        if (winner === 'BLACKJACK') {
            setChips(prev => prev + currentBet * 2.5);
        } else {
            setChips(prev => prev + chipChange);
        }

        setCurrentBet(0);

        if (finalWinner === 'PLAYER') {
            setMessage(Kazandınız! (${pVal} > ${dVal}). Kazanç: ${currentBet});
        } else if (winner === 'BLACKJACK') {
            setMessage(BLACKJACK! Kazandınız! (1.5x ödeme). Kazanç: ${currentBet * 1.5});
        } else if (finalWinner === 'DEALER') {
            setMessage(Krupiye kazandı (${dVal} > ${pVal}).);
        } else {
            setMessage(Berabere (Push). Bahis geri iade edildi.);
        }

    }, [currentBet, winner]);

    // Krupiye Sırası İzleyicisi
    useEffect(() => {
        if (gameState === 'DEALER_TURN') {
            // Kısa bir gecikme ekle
            const timer = setTimeout(() => {
                dealerPlay();
            }, 1000);
            return () => clearTimeout(timer);
        }
    }, [gameState, dealerPlay]);

    // Oyun Bitti, Yeniden Başlatma
    const resetGame = () => {
        setPlayerHand([]);
        setDealerHand([]);
        setCurrentBet(0);
        setBet(50);
        setWinner(null);
        setDeck(createAndShuffleDeck());
        setGameState('BETTING');
        setMessage('Yeni tur için bahis yapın.');
    };

    // ===========================================
    // UI - BİLEŞENLER VE GÖSTERİMLER
    // ===========================================

    // Hand Renders
    const renderDealerHand = () => (
        <div className="flex justify-center flex-wrap">
            {dealerHand.map((card, index) => (
                <Card key={index} card={card} hidden={gameState !== 'RESULT' && index === 1} />
            ))}
        </div>
    );

    const renderPlayerHand = () => (
        <div className="flex justify-center flex-wrap">
            {playerHand.map((card, index) => (
                <Card key={index} card={card} />
            ))}
        </div>
    );

Kral Şengül, [27.11.2025 10:16]
// Ana Aksiyon Butonu
    const ActionButton = ({ onClick, text, disabled = false, className = '' }) => (
        <button
            onClick={onClick}
            disabled={disabled}
            className={py-3 px-6 rounded-lg font-semibold text-white transition duration-200 shadow-md transform hover:scale-105 active:scale-95 ${disabled ? 'bg-gray-400 cursor-not-allowed' : 'bg-green-600 hover:bg-green-700'} ${className}}
        >
            {text}
        </button>
    );

    // Bahis Kontrolü
    const BetControls = () => (
        <div className="flex flex-col space-y-4 items-center">
            <div className="flex space-x-2">
                <ActionButton 
                    text="50" 
                    onClick={() => placeBet(50)} 
                    disabled={chips < 50}
                    className={bet === 50 ? 'bg-yellow-500 hover:bg-yellow-600' : 'bg-indigo-500 hover:bg-indigo-600'}
                />
                <ActionButton 
                    text="100" 
                    onClick={() => placeBet(100)} 
                    disabled={chips < 100}
                    className={bet === 100 ? 'bg-yellow-500 hover:bg-yellow-600' : 'bg-indigo-500 hover:bg-indigo-600'}
                />
                <ActionButton 
                    text="200" 
                    onClick={() => placeBet(200)} 
                    disabled={chips < 200}
                    className={bet === 200 ? 'bg-yellow-500 hover:bg-yellow-600' : 'bg-indigo-500 hover:bg-indigo-600'}
                />
            </div>
            <ActionButton 
                onClick={() => {
                    setCurrentBet(bet);
                    dealInitialCards();
                }}
                text={DAĞIT (Bahis: ${bet})}
                disabled={bet <= 0 || chips < bet}
                className="bg-red-600 w-full max-w-xs"
            />
        </div>
    );

    return (
        <div className="min-h-screen bg-gray-900 flex flex-col items-center justify-start py-8 px-4 font-sans text-white">
            <style>
                {/* Tailwind'in mobil uyumluluk için varsayılan fontunu kullanıyoruz */}
                {
                .chip-display {
                    background: linear-gradient(145deg, #1f2937, #374151);
                    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.4);
                }
                }
            </style>
            
            <header className="w-full max-w-xl text-center mb-6">
                <h1 className="text-4xl font-bold tracking-tight text-yellow-400">
                    🃏 Blackjack Mini App
                </h1>
                <div className="chip-display p-3 mt-4 rounded-xl text-lg font-bold flex justify-between items-center">
                    <span>Çipleriniz: ${chips}</span>
                    <span className="text-green-400">Mevcut Bahis: ${currentBet}</span>
                </div>
            </header>

            <main className="w-full max-w-xl space-y-6">
                
                {/* Mesaj Kutusu */}
                <div className={p-4 rounded-lg text-center font-semibold text-xl ${winner ? (winner === 'PLAYER' || winner === 'BLACKJACK' ? 'bg-yellow-600 text-white' : winner === 'PUSH' ? 'bg-blue-500 text-white' : 'bg-red-600 text-white') : 'bg-gray-700 text-gray-200'}}>
                    {message}
                </div>

                {/* Krupiye Alanı */}
                <section className="bg-gray-800 p-4 rounded-xl shadow-inner space-y-3">
                    <h2 className="text-2xl font-semibold border-b border-gray-600 pb-1 flex justify-between items-center">
                        Krupiye ({gameState === 'RESULT' ? dealerValue : dealerVisibleValue})
                        <span className="text-sm text-gray-400">{gameState !== 'RESULT' && dealerHand.length > 0 && Görünür: ${dealerVisibleValue}}</span>

Kral Şengül, [27.11.2025 10:16]
</h2>
                    {renderDealerHand()}
                </section>
                
                {/* Oyuncu Alanı */}
                <section className="bg-gray-800 p-4 rounded-xl shadow-inner space-y-3">
                    <h2 className="text-2xl font-semibold border-b border-gray-600 pb-1">
                        Siz ({playerValue})
                    </h2>
                    {renderPlayerHand()}
                </section>

                {/* Kontrol Alanı */}
                <section className="p-4 bg-gray-700 rounded-xl">
                    {gameState === 'BETTING' && <BetControls />}

                    {gameState === 'PLAYER_TURN' && (
                        <div className="flex justify-center space-x-3">
                            <ActionButton 
                                onClick={handleHit} 
                                text="Vur (Hit)"
                                className="bg-blue-600"
                            />
                            <ActionButton 
                                onClick={handleStand} 
                                text="Dur (Stand)"
                                className="bg-red-600"
                            />
                             <ActionButton 
                                onClick={handleDoubleDown} 
                                text="İkiye Katla"
                                disabled={playerHand.length !== 2 || chips < currentBet}
                                className="bg-orange-500"
                            />
                        </div>
                    )}
                    
                    {gameState === 'DEALER_TURN' && (
                        <div className="text-center py-2 text-xl font-bold text-yellow-500 animate-pulse">
                            KRUPİYE OYNUYOR...
                        </div>
                    )}

                    {gameState === 'RESULT' && (
                        <div className="text-center">
                            <ActionButton 
                                onClick={resetGame} 
                                text="Yeni Oyun Başlat"
                                className="bg-purple-600 w-full max-w-xs"
                            />
                        </div>
                    )}
                </section>
            </main>
            
            {/* Alt Bilgi */}
            <footer className="mt-8 text-center text-sm text-gray-500">
                <p>Oyun Kuralları: Krupiye 17'de durur. As (Ace) 1 veya 11 olarak sayılır.</p>
            </footer>
        </div>
    );
};

export default App;
