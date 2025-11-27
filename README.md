# Blackjack-mini-app
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
