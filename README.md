import React, { useState, useEffect, useCallback } from 'react';
import { Play, Zap, Users, Trophy, Gift, TrendingUp, Heart, DollarSign, Star, Award, Sparkles } from 'lucide-react';

const CapybaraParadise = () => {
  // Game State
  const [happiness, setHappiness] = useState(0);
  const [happinessPerClick, setHappinessPerClick] = useState(1);
  const [happinessPerSecond, setHappinessPerSecond] = useState(0);
  const [capybaras, setCapybaras] = useState({
    baby: 0,
    adult: 0,
    elder: 0,
    rainbow: 0
  });
  
  // Upgrades
  const [upgrades, setUpgrades] = useState({
    petPower: 1,
    foodQuality: 1,
    playTime: 1
  });
  
  // Game Progress
  const [level, setLevel] = useState(1);
  const [prestige, setPrestige] = useState(0);
  const [achievements, setAchievements] = useState([]);
  const [totalHappiness, setTotalHappiness] = useState(0);
  
  // UI State
  const [showAdBoost, setShowAdBoost] = useState(false);
  const [adBoostActive, setAdBoostActive] = useState(false);
  const [adBoostTimeLeft, setAdBoostTimeLeft] = useState(0);
  const [tab, setTab] = useState('play');
  const [clickAnimation, setClickAnimation] = useState(false);
  
  // Mock Leaderboard Data
  const [leaderboard] = useState([
    { name: 'CapyLover', happiness: 1250000, level: 15, avatar: '🐹' },
    { name: 'FluffyFriend', happiness: 980000, level: 12, avatar: '🌸' },
    { name: 'ChillVibes', happiness: 750000, level: 10, avatar: '🍃' },
    { name: 'You', happiness: 0, level: 1, avatar: '⭐' },
    { name: 'NatureKid', happiness: 450000, level: 8, avatar: '🌿' }
  ]);

  // Capybara Types
  const capybaraTypes = {
    baby: { name: 'Baby Capybara', baseCost: 15, baseHappiness: 0.1, icon: '🐹', description: 'Tiny and adorable!' },
    adult: { name: 'Adult Capybara', baseCost: 100, baseHappiness: 1, icon: '🦫', description: 'Calm and peaceful' },
    elder: { name: 'Elder Capybara', baseCost: 1100, baseHappiness: 8, icon: '🧓🐹', description: 'Wise and serene' },
    rainbow: { name: 'Rainbow Capybara', baseCost: 12000, baseHappiness: 47, icon: '🌈🐹', description: 'Magical and rare!' }
  };

  // Upgrade Types
  const upgradeTypes = {
    petPower: { name: 'Gentle Petting', baseCost: 10, effect: 1, icon: '🤗', description: 'Pet more lovingly!' },
    foodQuality: { name: 'Yummy Treats', baseCost: 50, effect: 0.1, icon: '🥕', description: 'Better snacks!' },
    playTime: { name: 'Fun Activities', baseCost: 500, effect: 2, icon: '🎾', description: 'More playtime!' }
  };

  // Achievement System
  const achievementList = [
    { id: 'first_pet', name: 'First Pet', desc: 'Pet a capybara for the first time', requirement: 1, type: 'clicks', reward: '🐹' },
    { id: 'hundred_pets', name: 'Pet Master', desc: 'Pet capybaras 100 times', requirement: 100, type: 'clicks', reward: '🏆' },
    { id: 'first_capybara', name: 'New Friend', desc: 'Adopt your first capybara', requirement: 1, type: 'capybaras', reward: '❤️' },
    { id: 'happy_place', name: 'Happy Place', desc: 'Reach 10,000 happiness', requirement: 10000, type: 'total_happiness', reward: '🌟' },
    { id: 'paradise', name: 'Capybara Paradise', desc: 'Reach 1,000,000 happiness', requirement: 1000000, type: 'total_happiness', reward: '🏰' }
  ];

  // Cute messages that appear when clicking
  const clickMessages = [
    "So cute! 🥰", "Adorable! 💕", "Sweet! 🍯", "Aww! 😍", "Precious! 💎", 
    "Fluffy! ☁️", "Peaceful! 🕊️", "Lovely! 🌸", "Happy! 😊", "Cozy! 🏠"
  ];

  // Auto-save and Load
  useEffect(() => {
    const savedGame = localStorage.getItem('capybaraParadise');
    if (savedGame) {
      try {
        const gameState = JSON.parse(savedGame);
        setHappiness(gameState.happiness || 0);
        setHappinessPerClick(gameState.happinessPerClick || 1);
        setHappinessPerSecond(gameState.happinessPerSecond || 0);
        setCapybaras(gameState.capybaras || { baby: 0, adult: 0, elder: 0, rainbow: 0 });
        setUpgrades(gameState.upgrades || { petPower: 1, foodQuality: 1, playTime: 1 });
        setLevel(gameState.level || 1);
        setPrestige(gameState.prestige || 0);
        setAchievements(gameState.achievements || []);
        setTotalHappiness(gameState.totalHappiness || 0);
      } catch (e) {
        console.log('Error loading game state');
      }
    }
  }, []);

  // Auto-save
  useEffect(() => {
    const gameState = {
      happiness, happinessPerClick, happinessPerSecond, capybaras, upgrades,
      level, prestige, achievements, totalHappiness
    };
    localStorage.setItem('capybaraParadise', JSON.stringify(gameState));
  }, [happiness, happinessPerClick, happinessPerSecond, capybaras, upgrades, level, prestige, achievements, totalHappiness]);

  // Passive Happiness Timer
  useEffect(() => {
    const interval = setInterval(() => {
      if (happinessPerSecond > 0) {
        const earned = happinessPerSecond * (adBoostActive ? 2 : 1);
        setHappiness(prev => prev + earned);
        setTotalHappiness(prev => prev + earned);
      }
    }, 1000);
    return () => clearInterval(interval);
  }, [happinessPerSecond, adBoostActive]);

  // Ad Boost Timer
  useEffect(() => {
    if (adBoostActive && adBoostTimeLeft > 0) {
      const timer = setTimeout(() => {
        setAdBoostTimeLeft(prev => prev - 1);
      }, 1000);
      return () => clearTimeout(timer);
    } else if (adBoostTimeLeft === 0) {
      setAdBoostActive(false);
    }
  }, [adBoostActive, adBoostTimeLeft]);

  // Calculate costs
  const getCapybaraCost = (type) => {
    return Math.floor(capybaraTypes[type].baseCost * Math.pow(1.15, capybaras[type]));
  };

  const getUpgradeCost = (type) => {
    return Math.floor(upgradeTypes[type].baseCost * Math.pow(1.5, upgrades[type] - 1));
  };

  // Main click handler
  const handleClick = () => {
    const earned = happinessPerClick * (adBoostActive ? 2 : 1);
    setHappiness(prev => prev + earned);
    setTotalHappiness(prev => prev + earned);
    setClickAnimation(true);
    
    // Show random cute message
    const message = clickMessages[Math.floor(Math.random() * clickMessages.length)];
    
    setTimeout(() => setClickAnimation(false), 200);
    checkAchievements('clicks', 1);
  };

  // Adopt capybara
  const adoptCapybara = (type) => {
    const cost = getCapybaraCost(type);
    if (happiness >= cost) {
      setHappiness(prev => prev - cost);
      setCapybaras(prev => ({ ...prev, [type]: prev[type] + 1 }));
      
      // Update happiness per second
      const newHappiness = capybaraTypes[type].baseHappiness * upgrades.foodQuality;
      setHappinessPerSecond(prev => prev + newHappiness);
      
      checkAchievements('capybaras', 1);
    }
  };

  // Buy upgrade
  const buyUpgrade = (type) => {
    const cost = getUpgradeCost(type);
    if (happiness >= cost) {
      setHappiness(prev => prev - cost);
      setUpgrades(prev => ({ ...prev, [type]: prev[type] + 1 }));
      
      if (type === 'petPower') {
        setHappinessPerClick(prev => prev + upgradeTypes[type].effect);
      } else if (type === 'foodQuality') {
        // Recalculate all capybara happiness
        let totalHappinessRate = 0;
        Object.keys(capybaras).forEach(capyType => {
          totalHappinessRate += capybaras[capyType] * capybaraTypes[capyType].baseHappiness * (upgrades.foodQuality + upgradeTypes[type].effect);
        });
        setHappinessPerSecond(totalHappinessRate);
      }
    }
  };

  // Watch Ad for Boost
  const watchAdForBoost = () => {
    alert('🎬 Video watched! Your capybaras are extra happy for 30 minutes! 🌟');
    setAdBoostActive(true);
    setAdBoostTimeLeft(1800); // 30 minutes
    setShowAdBoost(false);
  };

  // Check achievements
  const checkAchievements = (type, value) => {
    achievementList.forEach(achievement => {
      if (!achievements.includes(achievement.id)) {
        let shouldUnlock = false;
        
        switch (achievement.type) {
          case 'clicks':
            shouldUnlock = true;
            break;
          case 'capybaras':
            const totalCapybaras = Object.values(capybaras).reduce((a, b) => a + b, 0);
            shouldUnlock = totalCapybaras >= achievement.requirement;
            break;
          case 'total_happiness':
            shouldUnlock = totalHappiness >= achievement.requirement;
            break;
        }
        
        if (shouldUnlock) {
          setAchievements(prev => [...prev, achievement.id]);
          setHappiness(prev => prev + achievement.requirement * 0.1);
        }
      }
    });
  };

  // Format numbers
  const formatNumber = (num) => {
    if (num >= 1000000000000) return (num / 1000000000000).toFixed(2) + 'T';
    if (num >= 1000000000) return (num / 1000000000).toFixed(2) + 'B';
    if (num >= 1000000) return (num / 1000000).toFixed(2) + 'M';
    if (num >= 1000) return (num / 1000).toFixed(2) + 'K';
    return num.toFixed(2);
  };

  // Format time
  const formatTime = (seconds) => {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = seconds % 60;
    if (hours > 0) return `${hours}h ${minutes}m`;
    return `${minutes}m ${secs}s`;
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-green-200 via-blue-200 to-purple-200 text-gray-800">
      {/* Header */}
      <div className="bg-white/80 backdrop-blur-sm border-b border-green-200 p-4 shadow-lg">
        <div className="max-w-6xl mx-auto flex justify-between items-center">
          <div className="flex items-center space-x-4">
            <div className="text-2xl font-bold flex items-center text-green-700">
              <Heart className="mr-2 text-pink-500" />
              🐹 Capybara Paradise
            </div>
            <div className="text-sm text-gray-600 bg-green-100 px-3 py-1 rounded-full">Level {level}</div>
          </div>
          
          <div className="flex items-center space-x-6">
            <div className="text-right">
              <div className="text-2xl font-bold text-pink-600 flex items-center">
                <Sparkles className="mr-1 w-6 h-6" />
                {formatNumber(happiness)}
              </div>
              <div className="text-sm text-gray-600">
                +{formatNumber(happinessPerSecond)}/sec happiness
              </div>
            </div>
            
            {adBoostActive && (
              <div className="bg-yellow-100 border-2 border-yellow-400 rounded-lg px-3 py-2 text-center animate-pulse">
                <div className="text-yellow-600 font-bold">🌟 EXTRA HAPPY!</div>
                <div className="text-xs text-yellow-600">{formatTime(adBoostTimeLeft)}</div>
              </div>
            )}
          </div>
        </div>
      </div>

      {/* Navigation */}
      <div className="bg-white/60 border-b border-green-200">
        <div className="max-w-6xl mx-auto flex space-x-1 p-2">
          {[
            { id: 'play', icon: Heart, label: 'Play', color: 'pink' },
            { id: 'family', icon: Users, label: 'Capybara Family', color: 'green' },
            { id: 'upgrades', icon: TrendingUp, label: 'Upgrades', color: 'blue' },
            { id: 'leaderboard', icon: Trophy, label: 'Friends', color: 'yellow' },
            { id: 'achievements', icon: Award, label: 'Achievements', color: 'purple' }
          ].map(tabItem => (
            <button
              key={tabItem.id}
              onClick={() => setTab(tabItem.id)}
              className={`flex items-center space-x-2 px-4 py-2 rounded-lg transition-all font-medium ${
                tab === tabItem.id 
                  ? `bg-${tabItem.color}-200 border-2 border-${tabItem.color}-400 text-${tabItem.color}-700` 
                  : 'hover:bg-white/80 text-gray-700'
              }`}
            >
              <tabItem.icon size={18} />
              <span>{tabItem.label}</span>
            </button>
          ))}
        </div>
      </div>

      <div className="max-w-6xl mx-auto p-6">
        {/* Kid-Friendly Ad Banner */}
        <div className="bg-gradient-to-r from-yellow-100 to-orange-100 border-2 border-yellow-300 rounded-lg p-3 mb-6 text-center">
          <div className="text-yellow-700 text-sm font-medium">🎨 Fun Educational Games - Safe for Kids!</div>
          <div className="text-xs text-yellow-600 mt-1">Child-Safe Ad Network Integration</div>
        </div>

        {tab === 'play' && (
          <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
            {/* Main Playing Area */}
            <div className="text-center">
              <div className="mb-6">
                <div className="text-xl font-bold text-green-700 mb-4">Pet the Capybara! 🐹</div>
                <button
                  onClick={handleClick}
                  className={`w-48 h-48 mx-auto bg-gradient-to-br from-yellow-300 to-orange-300 rounded-full shadow-2xl hover:scale-105 transform transition-all duration-200 flex items-center justify-center text-6xl hover:shadow-yellow-400/50 border-4 border-yellow-400 ${
                    clickAnimation ? 'animate-bounce' : ''
                  }`}
                >
                  🐹
                </button>
                <div className="mt-4">
                  <div className="text-lg text-gray-700">Happiness per Pet</div>
                  <div className="text-2xl font-bold text-pink-600 flex items-center justify-center">
                    <Sparkles className="mr-2 w-6 h-6" />
                    +{formatNumber(happinessPerClick * (adBoostActive ? 2 : 1))}
                  </div>
                </div>
              </div>

              {/* Kid-Friendly Reward Buttons */}
              <div className="space-y-4">
                <button
                  onClick={() => setShowAdBoost(true)}
                  className="w-full bg-gradient-to-r from-green-400 to-emerald-500 hover:from-green-500 hover:to-emerald-600 py-3 px-6 rounded-lg font-bold text-white flex items-center justify-center space-x-2 transition-all shadow-lg"
                  disabled={adBoostActive}
                >
                  <Gift className="w-5 h-5" />
                  <span>{adBoostActive ? '🌟 Extra Happy Time!' : '🎬 Watch Fun Video for Double Happiness!'}</span>
                </button>
                
                <button
                  onClick={() => alert('🎬 Fun video watched! Here are bonus happiness points! 🌟')}
                  className="w-full bg-gradient-to-r from-purple-400 to-pink-500 hover:from-purple-500 hover:to-pink-600 py-2 px-4 rounded-lg font-medium text-white flex items-center justify-center space-x-2 transition-all shadow-lg"
                >
                  <Sparkles className="w-4 h-4" />
                  <span>🎁 Free Happiness (Watch Video)</span>
                </button>
              </div>
            </div>

            {/* Capybara Family Status */}
            <div>
              <h2 className="text-2xl font-bold mb-4 flex items-center text-green-700">
                <Users className="mr-2" /> 🏠 Your Capybara Family
              </h2>
              <div className="space-y-3">
                {Object.entries(capybaraTypes).map(([type, capybara]) => (
                  <div key={type} className="bg-white/80 backdrop-blur-sm border-2 border-green-200 rounded-lg p-4 shadow-md">
                    <div className="flex justify-between items-center">
                      <div className="flex items-center space-x-3">
                        <div className="text-3xl">{capybara.icon}</div>
                        <div>
                          <div className="font-bold text-gray-800">{capybara.name}</div>
                          <div className="text-sm text-gray-600">{capybara.description}</div>
                          <div className="text-sm text-green-600 font-medium">
                            Family: {capybaras[type]} | Happiness: +{formatNumber(capybara.baseHappiness * upgrades.foodQuality)}/sec each
                          </div>
                        </div>
                      </div>
                      <button
                        onClick={() => adoptCapybara(type)}
                        disabled={happiness < getCapybaraCost(type)}
                        className="bg-green-500 hover:bg-green-600 disabled:bg-gray-400 disabled:text-gray-600 text-white px-4 py-2 rounded-lg font-medium transition-all shadow-md flex items-center space-x-1"
                      >
                        <Sparkles className="w-4 h-4" />
                        <span>{formatNumber(getCapybaraCost(type))}</span>
                      </button>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        )}

        {tab === 'family' && (
          <div>
            <h2 className="text-2xl font-bold mb-6 flex items-center text-green-700">
              <Users className="mr-2" /> 🏡 Capybara Family Management
            </h2>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
              {Object.entries(capybaraTypes).map(([type, capybara]) => (
                <div key={type} className="bg-white/80 backdrop-blur-sm border-2 border-green-200 rounded-lg p-6 shadow-lg">
                  <div className="text-center">
                    <div className="text-6xl mb-3">{capybara.icon}</div>
                    <h3 className="text-xl font-bold text-gray-800 mb-2">{capybara.name}</h3>
                    <p className="text-gray-600 mb-2">{capybara.description}</p>
                    <div className="bg-green-100 rounded-lg p-3 mb-4">
                      <div className="text-green-700 font-bold">Family Size: {capybaras[type]}</div>
                      <div className="text-sm text-green-600">Total Happiness: +{formatNumber(capybaras[type] * capybara.baseHappiness * upgrades.foodQuality)}/sec</div>
                    </div>
                    <button
                      onClick={() => adoptCapybara(type)}
                      disabled={happiness < getCapybaraCost(type)}
                      className="w-full bg-gradient-to-r from-green-500 to-emerald-600 hover:from-green-600 hover:to-emerald-700 disabled:from-gray-400 disabled:to-gray-500 text-white py-3 px-4 rounded-lg font-bold transition-all shadow-md"
                    >
                      Adopt New Friend - ✨ {formatNumber(getCapybaraCost(type))}
                    </button>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {tab === 'upgrades' && (
          <div>
            <h2 className="text-2xl font-bold mb-6 flex items-center text-blue-700">
              <TrendingUp className="mr-2" /> 🎯 Happiness Upgrades
            </h2>
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
              {Object.entries(upgradeTypes).map(([type, upgrade]) => (
                <div key={type} className="bg-white/80 backdrop-blur-sm border-2 border-blue-200 rounded-lg p-6 shadow-lg">
                  <div className="text-center">
                    <div className="text-4xl mb-3">{upgrade.icon}</div>
                    <h3 className="text-xl font-bold text-gray-800 mb-2">{upgrade.name}</h3>
                    <p className="text-gray-600 mb-2">{upgrade.description}</p>
                    <div className="bg-blue-100 rounded-lg p-2 mb-4">
                      <div className="text-blue-700 font-bold">Level {upgrades[type]}</div>
                    </div>
                    <button
                      onClick={() => buyUpgrade(type)}
                      disabled={happiness < getUpgradeCost(type)}
                      className="w-full bg-gradient-to-r from-blue-500 to-purple-600 hover:from-blue-600 hover:to-purple-700 disabled:from-gray-400 disabled:to-gray-500 text-white py-3 px-4 rounded-lg font-bold transition-all shadow-md"
                    >
                      Upgrade - ✨ {formatNumber(getUpgradeCost(type))}
                    </button>
                  </div>
                </div>
              ))}
            </div>
          </div>
        )}

        {tab === 'leaderboard' && (
          <div>
            <h2 className="text-2xl font-bold mb-6 flex items-center text-yellow-700">
              <Trophy className="mr-2" /> 🏆 Happy Friends Leaderboard
            </h2>
            <div className="bg-white/80 backdrop-blur-sm border-2 border-yellow-200 rounded-lg overflow-hidden shadow-lg">
              {leaderboard.map((player, index) => (
                <div key={index} className={`flex items-center justify-between p-4 border-b border-yellow-200 last:border-b-0 ${
                  player.name === 'You' ? 'bg-yellow-100 border-yellow-400' : ''
                }`}>
                  <div className="flex items-center space-x-4">
                    <div className={`w-10 h-10 rounded-full flex items-center justify-center font-bold text-lg ${
                      index === 0 ? 'bg-yellow-400 text-white shadow-lg' :
                      index === 1 ? 'bg-gray-300 text-gray-800 shadow-md' :
                      index === 2 ? 'bg-amber-600 text-white shadow-md' :
                      'bg-gray-200 text-gray-600'
                    }`}>
                      {index < 3 ? ['🥇', '🥈', '🥉'][index] : (index + 1)}
                    </div>
                    <div>
                      <div className="font-bold text-gray-800 flex items-center">
                        {player.avatar} {player.name}
                      </div>
                      <div className="text-sm text-gray-600">Level {player.level}</div>
                    </div>
                  </div>
                  <div className="text-pink-600 font-bold flex items-center">
                    <Sparkles className="mr-1 w-4 h-4" />
                    {formatNumber(player.name === 'You' ? happiness : player.happiness)}
                  </div>
                </div>
              ))}
            </div>
            
            <div className="mt-6 bg-gradient-to-r from-green-100 to-blue-100 border-2 border-green-300 rounded-lg p-4 text-center shadow-lg">
              <div className="text-green-700 font-bold mb-2">🎉 Invite Friends to Play!</div>
              <div className="text-sm text-gray-700 mb-3">Share the fun and see who can make their capybaras happiest!</div>
              <button className="bg-green-500 hover:bg-green-600 text-white px-6 py-2 rounded-lg font-medium transition-all shadow-md">
                📱 Share with Friends
              </button>
            </div>
          </div>
        )}

        {tab === 'achievements' && (
          <div>
            <h2 className="text-2xl font-bold mb-6 flex items-center text-purple-700">
              <Award className="mr-2" /> 🏅 Achievements
            </h2>
            <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
              {achievementList.map(achievement => {
                const isUnlocked = achievements.includes(achievement.id);
                return (
                  <div key={achievement.id} className={`bg-white/80 backdrop-blur-sm border-2 rounded-lg p-4 shadow-lg ${
                    isUnlocked ? 'border-yellow-400 bg-yellow-50' : 'border-purple-200'
                  }`}>
                    <div className="flex items-center space-x-3">
                      <div className={`w-12 h-12 rounded-full flex items-center justify-center text-2xl ${
                        isUnlocked ? 'bg-yellow-400 text-white shadow-lg' : 'bg-gray-200 text-gray-500'
                      }`}>
                        {isUnlocked ? achievement.reward : '🔒'}
                      </div>
                      <div>
                        <div className={`font-bold ${isUnlocked ? 'text-yellow-700' : 'text-gray-600'}`}>
                          {achievement.name}
                        </div>
                        <div className="text-sm text-gray-600">{achievement.desc}</div>
                        {isUnlocked && (
                          <div className="text-xs text-green-600 mt-1 font-medium">🎉 Completed!</div>
                        )}
                      </div>
                    </div>
                  </div>
                );
              })}
            </div>
          </div>
        )}
      </div>

      {/* Ad Boost Modal - Kid Friendly */}
      {showAdBoost && (
        <div className="fixed inset-0 bg-black/30 backdrop-blur-sm flex items-center justify-center z-50 p-4">
          <div className="bg-white border-2 border-green-300 rounded-lg p-6 max-w-sm w-full shadow-2xl">
            <h3 className="text-xl font-bold mb-4 text-center text-green-700">Watch Fun Video! 🎬</h3>
            <div className="text-center mb-6">
              <div className="text-6xl mb-3">🌟</div>
              <p className="text-gray-700 mb-2">Watch a fun, educational video to help your capybaras!</p>
              <div className="bg
