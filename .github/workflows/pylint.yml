# main.py - Complete MarketEdge Analyzer Application
import streamlit as st
import pandas as pd
import numpy as np
import yfinance as yf
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import ta  # Technical analysis library
from datetime import datetime, timedelta
import requests
import json
from typing import List, Dict, Tuple
import warnings
warnings.filterwarnings('ignore')

# ==================== CONFIGURATION ====================
st.set_page_config(
    page_title="MarketEdge Analyzer",
    page_icon="📈",
    layout="wide",
    initial_sidebar_state="expanded"
)

# ==================== CUSTOM CSS ====================
st.markdown("""

    .main-header {
        font-size: 2.5rem;
        color: #2E86AB;
        font-weight: 700;
    }
    .sub-header {
        font-size: 1.5rem;
        color: #1C3F60;
        font-weight: 600;
        margin-top: 20px;
    }
    .card {
        padding: 20px;
        border-radius: 10px;
        box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
        margin: 10px 0;
    }
    .positive {
        color: #00C853;
        font-weight: bold;
    }
    .negative {
        color: #FF5252;
        font-weight: bold;
    }
    .stock-card {
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        color: white;
        padding: 15px;
        border-radius: 10px;
        margin: 10px 0;
    }
    .metric-card {
        background: #f8f9fa;
        padding: 15px;
        border-radius: 8px;
        text-align: center;
        border-left: 4px solid #2E86AB;
    }
    .tab-content {
        animation: fadeIn 0.5s;
    }
    @keyframes fadeIn {
        from { opacity: 0; }
        to { opacity: 1; }
    }

""", unsafe_allow_html=True)

# ==================== INITIALIZATION ====================
class MarketEdgeAnalyzer:
    def __init__(self):
        self.api_key = "demo"  # Replace with actual API key
        self.intraday_stocks = []
        self.longterm_stocks = []
        self.market_data = {}
        
    def fetch_market_data(self):
        """Fetch real-time market data"""
        try:
            indices = {
                'SPY': 'S&P 500',
                'QQQ': 'Nasdaq 100',
                'DIA': 'Dow Jones',
                'IWM': 'Russell 2000',
                '^VIX': 'VIX'
            }
            
            market_data = {}
            for symbol, name in indices.items():
                try:
                    ticker = yf.Ticker(symbol)
                    hist = ticker.history(period='5d')
                    if not hist.empty:
                        current = hist['Close'].iloc[-1]
                        prev = hist['Close'].iloc[-2]
                        change = ((current - prev) / prev) * 100
                        market_data[symbol] = {
                            'name': name,
                            'price': current,
                            'change': change,
                            'volume': hist['Volume'].iloc[-1] if 'Volume' in hist.columns else 0
                        }
                except:
                    continue
            
            self.market_data = market_data
            return market_data
        except Exception as e:
            st.error(f"Error fetching market data: {str(e)}")
            return {}
    
    def calculate_technical_indicators(self, df):
        """Calculate technical indicators for analysis"""
        df = df.copy()
        
        # Moving Averages
        df['SMA_20'] = ta.trend.sma_indicator(df['Close'], window=20)
        df['SMA_50'] = ta.trend.sma_indicator(df['Close'], window=50)
        df['EMA_12'] = ta.trend.ema_indicator(df['Close'], window=12)
        df['EMA_26'] = ta.trend.ema_indicator(df['Close'], window=26)
        
        # RSI
        df['RSI'] = ta.momentum.rsi(df['Close'], window=14)
        
        # MACD
        macd = ta.trend.MACD(df['Close'])
        df['MACD'] = macd.macd()
        df['MACD_Signal'] = macd.macd_signal()
        df['MACD_Diff'] = macd.macd_diff()
        
        # Bollinger Bands
        bb = ta.volatility.BollingerBands(df['Close'], window=20, window_dev=2)
        df['BB_Upper'] = bb.bollinger_hband()
        df['BB_Lower'] = bb.bollinger_lband()
        df['BB_Middle'] = bb.bollinger_mavg()
        
        # ATR for volatility
        df['ATR'] = ta.volatility.average_true_range(df['High'], df['Low'], df['Close'], window=14)
        
        # Volume indicators
        df['Volume_SMA'] = df['Volume'].rolling(window=20).mean()
        df['Volume_Ratio'] = df['Volume'] / df['Volume_SMA']
        
        return df
    
    def screen_intraday_stocks(self):
        """Screen for intraday trading opportunities"""
        intraday_candidates = []
        
        # Pre-defined watchlist with rationale
        watchlist = [
            {
                'symbol': 'SPY',
                'name': 'SPDR S&P 500 ETF',
                'reason': 'High liquidity, tight spreads, follows overall market',
                'category': 'ETF',
                'avg_volume': 80000000,
                'avg_volatility': 1.2
            },
            {
                'symbol': 'QQQ',
                'name': 'Invesco QQQ Trust',
                'reason': 'Tech-heavy, high volatility, good for momentum plays',
                'category': 'ETF',
                'avg_volume': 50000000,
                'avg_volatility': 1.5
            },
            {
                'symbol': 'AAPL',
                'name': 'Apple Inc.',
                'reason': 'Massive liquidity, news-driven moves, option activity',
                'category': 'Technology',
                'avg_volume': 60000000,
                'avg_volatility': 2.1
            },
            {
                'symbol': 'NVDA',
                'name': 'NVIDIA Corporation',
                'reason': 'High beta stock, AI momentum, frequent large moves',
                'category': 'Semiconductors',
                'avg_volume': 45000000,
                'avg_volatility': 3.8
            },
            {
                'symbol': 'TSLA',
                'name': 'Tesla Inc.',
                'reason': 'Extreme volatility, retail interest, Elon-driven news',
                'category': 'Automotive',
                'avg_volume': 120000000,
                'avg_volatility': 4.5
            },
            {
                'symbol': 'AMD',
                'name': 'Advanced Micro Devices',
                'reason': 'High relative strength vs sector, earnings volatility',
                'category': 'Semiconductors',
                'avg_volume': 55000000,
                'avg_volatility': 3.2
            },
            {
                'symbol': 'MSFT',
                'name': 'Microsoft Corporation',
                'reason': 'Strong trend following, institutional favorite',
                'category': 'Technology',
                'avg_volume': 30000000,
                'avg_volatility': 1.8
            },
            {
                'symbol': 'IWM',
                'name': 'iShares Russell 2000 ETF',
                'reason': 'Small cap volatility, economic sensitivity',
                'category': 'ETF',
                'avg_volume': 35000000,
                'avg_volatility': 1.8
            }
        ]
        
        # Analyze each stock
        for stock in watchlist:
            try:
                ticker = yf.Ticker(stock['symbol'])
                hist = ticker.history(period='5d', interval='1h')
                
                if len(hist) > 10:
                    # Calculate metrics
                    recent_close = hist['Close'].iloc[-1]
                    prev_close = hist['Close'].iloc[-2] if len(hist) > 1 else recent_close
                    daily_change = ((recent_close - prev_close) / prev_close) * 100
                    
                    # Get RSI
                    hist_with_tech = self.calculate_technical_indicators(hist)
                    current_rsi = hist_with_tech['RSI'].iloc[-1] if 'RSI' in hist_with_tech.columns else 50
                    
                    # Volume analysis
                    avg_volume = hist['Volume'].mean()
                    volume_ratio = avg_volume / stock['avg_volume'] if stock['avg_volume'] > 0 else 1
                    
                    # Determine signals
                    signals = []
                    if abs(daily_change) > 1.5:
                        signals.append(f"Active Move ({daily_change:.1f}%)")
                    if current_rsi < 30:
                        signals.append("RSI Oversold")
                    elif current_rsi > 70:
                        signals.append("RSI Overbought")
                    if volume_ratio > 1.5:
                        signals.append("High Volume")
                    
                    intraday_candidates.append({
                        **stock,
                        'current_price': recent_close,
                        'daily_change': daily_change,
                        'rsi': current_rsi,
                        'volume_ratio': volume_ratio,
                        'signals': signals,
                        'momentum_score': self.calculate_momentum_score(hist)
                    })
                    
            except Exception as e:
                continue
        
        # Sort by momentum score (highest first)
        intraday_candidates.sort(key=lambda x: x['momentum_score'], reverse=True)
        self.intraday_stocks = intraday_candidates
        return intraday_candidates
    
    def screen_longterm_stocks(self):
        """Screen for long-term investment opportunities"""
        longterm_candidates = []
        
        # Fundamentally strong companies
        watchlist = [
            {
                'symbol': 'JNJ',
                'name': 'Johnson & Johnson',
                'sector': 'Healthcare',
                'reason': 'Dividend Aristocrat, defensive stock, strong pipeline',
                'dividend_yield': 3.2,
                'pe_ratio': 15.8,
                'debt_equity': 0.45
            },
            {
                'symbol': 'JPM',
                'name': 'JPMorgan Chase',
                'sector': 'Financials',
                'reason': 'Banking leader, strong capital position, good dividend',
                'dividend_yield': 2.8,
                'pe_ratio': 12.5,
                'debt_equity': 1.2
            },
            {
                'symbol': 'MSFT',
                'name': 'Microsoft',
                'sector': 'Technology',
                'reason': 'Cloud dominance, AI leadership, strong cash flow',
                'dividend_yield': 0.8,
                'pe_ratio': 35.2,
                'debt_equity': 0.65
            },
            {
                'symbol': 'GOOGL',
                'name': 'Alphabet (Google)',
                'sector': 'Technology',
                'reason': 'Search monopoly, cloud growth, AI innovation',
                'dividend_yield': 0.0,
                'pe_ratio': 28.5,
                'debt_equity': 0.12
            },
            {
                'symbol': 'HD',
                'name': 'Home Depot',
                'sector': 'Consumer Discretionary',
                'reason': 'Dividend Aristocrat, housing market leader, ROIC > 40%',
                'dividend_yield': 2.5,
                'pe_ratio': 24.3,
                'debt_equity': 13.5
            },
            {
                'symbol': 'PG',
                'name': 'Procter & Gamble',
                'sector': 'Consumer Staples',
                'reason': '67 years of dividend increases, brand power, stable',
                'dividend_yield': 2.4,
                'pe_ratio': 26.8,
                'debt_equity': 0.68
            },
            {
                'symbol': 'UNH',
                'name': 'UnitedHealth Group',
                'sector': 'Healthcare',
                'reason': 'Healthcare leader, consistent growth, strong management',
                'dividend_yield': 1.6,
                'pe_ratio': 21.4,
                'debt_equity': 0.65
            },
            {
                'symbol': 'V',
                'name': 'Visa Inc.',
                'sector': 'Financials',
                'reason': 'Payment network monopoly, high margins, global growth',
                'dividend_yield': 0.8,
                'pe_ratio': 31.2,
                'debt_equity': 0.50
            }
        ]
        
        # Analyze each stock
        for stock in watchlist:
            try:
                ticker = yf.Ticker(stock['symbol'])
                info = ticker.info
                
                # Get fundamentals
                current_price = info.get('currentPrice', info.get('regularMarketPrice', 0))
                market_cap = info.get('marketCap', 0)
                
                # Calculate quality score
                quality_score = self.calculate_quality_score(stock, info)
                
                # Get historical performance
                hist = ticker.history(period='1y')
                if not hist.empty:
                    yearly_return = ((hist['Close'].iloc[-1] - hist['Close'].iloc[0]) / hist['Close'].iloc[0]) * 100
                    max_drawdown = self.calculate_max_drawdown(hist['Close'])
                else:
                    yearly_return = 0
                    max_drawdown = 0
                
                longterm_candidates.append({
                    **stock,
                    'current_price': current_price,
                    'market_cap': market_cap,
                    'quality_score': quality_score,
                    'yearly_return': yearly_return,
                    'max_drawdown': max_drawdown,
                    'risk_reward': self.calculate_risk_reward(quality_score, max_drawdown)
                })
                
            except Exception as e:
                continue
        
        # Sort by quality score (highest first)
        longterm_candidates.sort(key=lambda x: x['quality_score'], reverse=True)
        self.longterm_stocks = longterm_candidates
        return longterm_candidates
    
    def calculate_momentum_score(self, df):
        """Calculate momentum score for intraday stocks"""
        if len(df) < 20:
            return 50
        
        recent_returns = (df['Close'].iloc[-1] / df['Close'].iloc[-5] - 1) * 100 if len(df) >= 5 else 0
        volume_trend = df['Volume'].iloc[-5:].mean() / df['Volume'].iloc[-10:-5].mean() if len(df) >= 10 else 1
        
        # Combine factors
        score = 50  # Base score
        score += min(20, max(-20, recent_returns))  # Returns component
        score += 10 if volume_trend > 1.2 else -10  # Volume component
        
        return max(0, min(100, score))
    
    def calculate_quality_score(self, stock, info):
        """Calculate quality score for long-term stocks"""
        score = 50  # Base score
        
        # Earnings quality
        if 'profitMargins' in info:
            score += info['profitMargins'] * 10
        
        # Growth consistency
        if 'revenueGrowth' in info:
            score += min(20, info['revenueGrowth'] * 100)
        
        # Financial strength (lower debt is better)
        if stock['debt_equity'] < 1:
            score += 15
        elif stock['debt_equity'] < 2:
            score += 5
        
        # Valuation (lower P/E is better, but not too low)
        if 10 < stock['pe_ratio'] < 20:
            score += 15
        elif 20 < stock['pe_ratio'] < 30:
            score += 10
        elif stock['pe_ratio'] < 10:
            score += 5
        
        # Dividend (if applicable)
        if stock['dividend_yield'] > 2:
            score += 10
        
        return min(100, max(0, score))
    
    def calculate_max_drawdown(self, prices):
        """Calculate maximum drawdown"""
        peak = prices.expanding(min_periods=1).max()
        drawdown = (prices - peak) / peak
        return drawdown.min() * 100
    
    def calculate_risk_reward(self, quality_score, max_drawdown):
        """Calculate risk-reward ratio"""
        if max_drawdown == 0:
            return 0
        return quality_score / abs(max_drawdown)
    
    def create_stock_chart(self, symbol, period='1mo', interval='1d'):
        """Create interactive stock chart with indicators"""
        ticker = yf.Ticker(symbol)
        hist = ticker.history(period=period, interval=interval)
        
        if hist.empty:
            return None
        
        # Add technical indicators
        hist = self.calculate_technical_indicators(hist)
        
        # Create subplot figure
        fig = make_subplots(
            rows=3, cols=1,
            shared_xaxes=True,
            vertical_spacing=0.05,
            row_heights=[0.6, 0.2, 0.2],
            subplot_titles=(f'{symbol} Price & Indicators', 'Volume', 'RSI')
        )
        
        # Candlestick chart
        fig.add_trace(
            go.Candlestick(
                x=hist.index,
                open=hist['Open'],
                high=hist['High'],
                low=hist['Low'],
                close=hist['Close'],
                name='Price',
                showlegend=False
            ),
            row=1, col=1
        )
        
        # Moving averages
        fig.add_trace(
            go.Scatter(
                x=hist.index,
                y=hist['SMA_20'],
                name='SMA 20',
                line=dict(color='orange', width=1)
            ),
            row=1, col=1
        )
        
        fig.add_trace(
            go.Scatter(
                x=hist.index,
                y=hist['SMA_50'],
                name='SMA 50',
                line=dict(color='blue', width=1)
            ),
            row=1, col=1
        )
        
        # Bollinger Bands
        fig.add_trace(
            go.Scatter(
                x=hist.index,
                y=hist['BB_Upper'],
                name='BB Upper',
                line=dict(color='gray', width=1, dash='dash'),
                opacity=0.5
            ),
            row=1, col=1
        )
        
        fig.add_trace(
            go.Scatter(
                x=hist.index,
                y=hist['BB_Lower'],
                name='BB Lower',
                line=dict(color='gray', width=1, dash='dash'),
                fill='tonexty',
                opacity=0.5
            ),
            row=1, col=1
        )
        
        # Volume
        colors = ['red' if hist['Close'].iloc[i] < hist['Close'].iloc[i-1] else 'green' 
                 for i in range(len(hist))]
        
        fig.add_trace(
            go.Bar(
                x=hist.index,
                y=hist['Volume'],
                name='Volume',
                marker_color=colors,
                showlegend=False
            ),
            row=2, col=1
        )
        
        # RSI
        fig.add_trace(
            go.Scatter(
                x=hist.index,
                y=hist['RSI'],
                name='RSI',
                line=dict(color='purple', width=2)
            ),
            row=3, col=1
        )
        
        # RSI bands
        fig.add_hline(y=70, line_dash="dash", line_color="red", opacity=0.5, row=3, col=1)
        fig.add_hline(y=30, line_dash="dash", line_color="green", opacity=0.5, row=3, col=1)
        
        fig.update_layout(
            height=700,
            title=f"{symbol} Technical Analysis",
            xaxis_rangeslider_visible=False,
            showlegend=True,
            legend=dict(
                orientation="h",
                yanchor="bottom",
                y=1.02,
                xanchor="right",
                x=1
            )
        )
        
        fig.update_xaxes(title_text="Date", row=3, col=1)
        fig.update_yaxes(title_text="Price ($)", row=1, col=1)
        fig.update_yaxes(title_text="Volume", row=2, col=1)
        fig.update_yaxes(title_text="RSI", row=3, col=1)
        
        return fig

# ==================== STREAMLIT APP ====================
def main():
    # Initialize analyzer
    analyzer = MarketEdgeAnalyzer()
    
    # Sidebar
    with st.sidebar:
        st.image("📈", width=100)
        st.title("MarketEdge Analyzer")
        st.markdown("---")
        
        # User Profile
        st.subheader("👤 Investor Profile")
        risk_profile = st.selectbox(
            "Risk Tolerance",
            ["Conservative", "Moderate", "Aggressive"],
            index=1
        )
        
        capital = st.number_input(
            "Investment Capital ($)",
            min_value=1000,
            max_value=1000000,
            value=10000,
            step=1000
        )
        
        investment_horizon = st.selectbox(
            "Investment Horizon",
            ["Intraday", "Swing (1-4 weeks)", "Medium (1-12 months)", "Long-term (>1 year)"],
            index=3
        )
        
        st.markdown("---")
        
        # Quick Actions
        st.subheader("⚡ Quick Actions")
        refresh_data = st.button("🔄 Refresh Market Data", use_container_width=True)
        view_intraday = st.button("📊 View Intraday Picks", use_container_width=True)
        view_longterm = st.button("🏆 View Long-term Picks", use_container_width=True)
        
        st.markdown("---")
        
        # Disclaimer
        st.warning("""
        **⚠️ Disclaimer:** This tool is for educational purposes only. 
        Not financial advice. Past performance doesn't guarantee future results.
        """)
    
    # Main Dashboard
    st.markdown('📈 MarketEdge Analyzer', unsafe_allow_html=True)
    st.markdown("Dual-strategy stock analysis for intraday trading & long-term investing")
    
    # Tabs
    tab1, tab2, tab3, tab4, tab5 = st.tabs([
        "📊 Market Overview", 
        "⚡ Intraday Scanner", 
        "🏆 Long-term Picks", 
        "📈 Stock Analyzer",
        "🛡️ Risk Management"
    ])
    
    # Tab 1: Market Overview
    with tab1:
        col1, col2, col3, col4, col5 = st.columns(5)
        
        # Fetch market data
        market_data = analyzer.fetch_market_data()
        
        # Display market indices
        indices = ['SPY', 'QQQ', 'DIA', 'IWM', '^VIX']
        for idx, symbol in enumerate(indices):
            with [col1, col2, col3, col4, col5][idx]:
                if symbol in market_data:
                    data = market_data[symbol]
                    change_class = "positive" if data['change'] >= 0 else "negative"
                    st.metric(
                        label=data['name'],
                        value=f"${data['price']:.2f}",
                        delta=f"{data['change']:.2f}%"
                    )
        
        # Market Summary
        st.markdown('Market Summary', unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            st.markdown('', unsafe_allow_html=True)
            st.subheader("Today's Outlook")
            
            # Simple market sentiment
            up_count = sum(1 for symbol in ['SPY', 'QQQ', 'DIA'] 
                          if symbol in market_data and market_data[symbol]['change'] > 0)
            
            if up_count >= 2:
                st.success("**Bullish Bias**: Majority of indices are positive")
                st.markdown("Market is showing strength. Consider looking for buying opportunities.")
            elif up_count == 1:
                st.info("**Mixed Market**: Indices are mixed")
                st.markdown("Market is consolidating. Be selective with positions.")
            else:
                st.error("**Bearish Bias**: Majority of indices are negative")
                st.markdown("Market is showing weakness. Consider defensive positions.")
            
            st.markdown('', unsafe_allow_html=True)
        
        with col2:
            st.markdown('', unsafe_allow_html=True)
            st.subheader("Key Levels to Watch")
            
            levels = {
                "S&P 500 Support": "4,600",
                "S&P 500 Resistance": "4,800",
                "Nasdaq Support": "16,000",
                "Nasdaq Resistance": "16,800",
                "VIX Alert Level": "> 18"
            }
            
            for level, value in levels.items():
                st.markdown(f"- **{level}**: {value}")
            
            st.markdown('', unsafe_allow_html=True)
        
        # Sector Performance (simulated)
        st.markdown('Sector Performance', unsafe_allow_html=True)
        
        sectors = {
            "Technology": {"change": 1.8, "trend": "up"},
            "Financials": {"change": 0.5, "trend": "up"},
            "Healthcare": {"change": -0.3, "trend": "down"},
            "Consumer Discretionary": {"change": 1.2, "trend": "up"},
            "Energy": {"change": -1.5, "trend": "down"}
        }
        
        cols = st.columns(len(sectors))
        for idx, (sector, data) in enumerate(sectors.items()):
            with cols[idx]:
                st.markdown(f'', unsafe_allow_html=True)
                st.markdown(f"**{sector}**")
                change_class = "positive" if data['change'] >= 0 else "negative"
                st.markdown(f'{data["change"]:.1f}%', unsafe_allow_html=True)
                st.markdown('', unsafe_allow_html=True)
    
    # Tab 2: Intraday Scanner
    with tab2:
        st.markdown('⚡ Intraday Trading Scanner', unsafe_allow_html=True)
        st.markdown("High liquidity, volatility-based opportunities for day trading")
        
        # Run scanner
        if st.button("Scan for Intraday Opportunities", type="primary"):
            with st.spinner("Scanning market for intraday opportunities..."):
                intraday_stocks = analyzer.screen_intraday_stocks()
                
                if intraday_stocks:
                    # Show top picks
                    st.markdown(f"### 🔥 Today's Top {len(intraday_stocks)} Intraday Picks")
                    
                    for i, stock in enumerate(intraday_stocks[:8]):  # Show top 8
                        col1, col2, col3, col4 = st.columns([1, 2, 2, 2])
                        
                        with col1:
                            st.markdown(f'', unsafe_allow_html=True)
                            st.markdown(f"**{stock['symbol']}**")
                            st.markdown(f"*{stock['category']}*")
                            st.markdown('', unsafe_allow_html=True)
                        
                        with col2:
                            st.markdown(f"**{stock['name']}**")
                            st.markdown(f"Price: ${stock['current_price']:.2f}")
                            change_class = "positive" if stock['daily_change'] >= 0 else "negative"
                            st.markdown(f'Change: {stock["daily_change"]:.2f}%', 
                                       unsafe_allow_html=True)
                        
                        with col3:
                            st.markdown("**Signals**")
                            for signal in stock['signals'][:3]:
                                st.markdown(f"• {signal}")
                        
                        with col4:
                            st.markdown("**Metrics**")
                            st.markdown(f"RSI: {stock['rsi']:.1f}")
                            st.markdown(f"Vol Ratio: {stock['volume_ratio']:.1f}x")
                            st.progress(stock['momentum_score'] / 100)
                            st.markdown(f"Momentum: {stock['momentum_score']:.0f}/100")
                        
                        st.markdown("---")
                    
                    # Strategy recommendations
                    st.markdown("### 🎯 Intraday Strategies")
                    
                    col1, col2, col3 = st.columns(3)
                    
                    with col1:
                        st.markdown('', unsafe_allow_html=True)
                        st.markdown("**Momentum Trading**")
                        st.markdown("• Trade with the trend")
                        st.markdown("• Use moving averages")
                        st.markdown("• Set tight stops")
                        st.markdown("*Best for: NVDA, AMD, TSLA*")
                        st.markdown('', unsafe_allow_html=True)
                    
                    with col2:
                        st.markdown('', unsafe_allow_html=True)
                        st.markdown("**Mean Reversion**")
                        st.markdown("• Fade extremes")
                        st.markdown("• RSI oversold/overbought")
                        st.markdown("• Scale in/out")
                        st.markdown("*Best for: SPY, QQQ, IWM*")
                        st.markdown('', unsafe_allow_html=True)
                    
                    with col3:
                        st.markdown('', unsafe_allow_html=True)
                        st.markdown("**Breakout Trading**")
                        st.markdown("• Enter on new highs/lows")
                        st.markdown("• Confirm with volume")
                        st.markdown("• Trail stops")
                        st.markdown("*Best for: AAPL, MSFT*")
                        st.markdown('', unsafe_allow_html=True)
                    
                    # Risk management
                    st.markdown("### 🛡️ Intraday Risk Management")
                    col1, col2, col3 = st.columns(3)
                    
                    with col1:
                        risk_per_trade = st.slider(
                            "Risk per Trade (% of Capital)",
                            min_value=0.5,
                            max_value=5.0,
                            value=1.0,
                            step=0.5
                        )
                    
                    with col2:
                        stop_loss = st.slider(
                            "Stop Loss (%)",
                            min_value=0.5,
                            max_value=10.0,
                            value=2.0,
                            step=0.5
                        )
                    
                    with col3:
                        take_profit = st.slider(
                            "Take Profit (%)",
                            min_value=1.0,
                            max_value=15.0,
                            value=4.0,
                            step=0.5
                        )
                    
                    st.info(f"For ${capital:,}: **Position size**: ${capital * risk_per_trade/100:.0f}, " +
                           f"**Stop loss**: ${stop_loss:.1f}%, **Target**: ${take_profit:.1f}%, **R/R**: 1:{take_profit/stop_loss:.1f}")
                
                else:
                    st.warning("No intraday opportunities found. Market may be closed or data unavailable.")
    
    # Tab 3: Long-term Picks
    with tab3:
        st.markdown('🏆 Long-term Investment Picks', unsafe_allow_html=True)
        st.markdown("Fundamentally strong companies for wealth building")
        
        if st.button("Generate Long-term Portfolio", type="primary"):
            with st.spinner("Analyzing long-term investment opportunities..."):
                longterm_stocks = analyzer.screen_longterm_stocks()
                
                if longterm_stocks:
                    # Portfolio Construction
                    st.markdown("### 📊 Suggested Portfolio Allocation")
                    
                    # Based on risk profile
                    portfolio = {
                        "Conservative": {"dividend": 60, "growth": 30, "defensive": 10},
                        "Moderate": {"dividend": 40, "growth": 50, "defensive": 10},
                        "Aggressive": {"dividend": 20, "growth": 70, "defensive": 10}
                    }
                    
                    allocation = portfolio.get(risk_profile, portfolio["Moderate"])
                    
                    col1, col2, col3 = st.columns(3)
                    
                    for i, (category, percent) in enumerate(allocation.items()):
                        st.markdown(f"**{category.capitalize()}**: {percent}% (${capital * percent/100:.0f})")
                    
                    # Show top picks
                    st.markdown(f"### 🏆 Top {len(longterm_stocks)} Long-term Picks")
                    
                    # Convert to DataFrame for better display
                    df_longterm = pd.DataFrame(longterm_stocks)
                    
                    # Create display columns
                    display_cols = ['symbol', 'name', 'sector', 'current_price', 
                                   'dividend_yield', 'pe_ratio', 'quality_score', 'yearly_return']
                    
                    st.dataframe(
                        df_longterm[display_cols].rename(columns={
                            'symbol': 'Symbol',
                            'name': 'Name',
                            'sector': 'Sector',
                            'current_price': 'Price ($)',
                            'dividend_yield': 'Div Yield (%)',
                            'pe_ratio': 'P/E Ratio',
                            'quality_score': 'Quality Score',
                            'yearly_return': '1Y Return (%)'
                        }),
                        use_container_width=True
                    )
                    
                    # Detailed Analysis
                    st.markdown("### 🔍 Detailed Analysis")
                    
                    for stock in longterm_stocks[:4]:  # Detailed look at top 4
                        with st.expander(f"📋 {stock['symbol']} - {stock['name']} (Quality: {stock['quality_score']}/100)"):
                            col1, col2 = st.columns(2)
                            
                            with col1:
                                st.markdown("**Investment Thesis**")
                                st.markdown(f"*{stock['reason']}*")
                                st.markdown("")
                                st.markdown("**Key Metrics**")
                                st.markdown(f"- Sector: {stock['sector']}")
                                st.markdown(f"- Price: ${stock['current_price']:.2f}")
                                st.markdown(f"- Dividend Yield: {stock['dividend_yield']:.1f}%")
                                st.markdown(f"- P/E Ratio: {stock['pe_ratio']:.1f}")
                                st.markdown(f"- Debt/Equity: {stock['debt_equity']:.1f}")
                            
                            with col2:
                                st.markdown("**Performance & Risk**")
                                st.markdown(f"- 1 Year Return: {stock['yearly_return']:.1f}%")
                                st.markdown(f"- Max Drawdown: {stock['max_drawdown']:.1f}%")
                                st.markdown(f"- Risk/Reward: {stock['risk_reward']:.1f}")
                                
                                # Simple rating
                                if stock['quality_score'] >= 80:
                                    st.success("⭐⭐⭐⭐⭐ Excellent Quality")
                                elif stock['quality_score'] >= 60:
                                    st.info("⭐⭐⭐⭐ Good Quality")
                                else:
                                    st.warning("⭐⭐⭐ Fair Quality")
                                
                                # Position size calculator
                                allocation_pct = st.slider(
                                    f"Allocation to {stock['symbol']} (%)",
                                    min_value=1.0,
                                    max_value=20.0,
                                    value=5.0,
                                    step=1.0,
                                    key=f"alloc_{stock['symbol']}"
                                )
                                
                                st.markdown(f"**Position Size**: ${capital * allocation_pct/100:.2f}")
                            
                            # Show chart
                            chart = analyzer.create_stock_chart(stock['symbol'], period='1y')
                            if chart:
                                st.plotly_chart(chart, use_container_width=True)
                    
                    # Portfolio Builder
                    st.markdown("### 🏗️ Portfolio Builder")
                    
                    selected_stocks = st.multiselect(
                        "Select stocks for your portfolio:",
                        options=[s['symbol'] for s in longterm_stocks],
                        default=[s['symbol'] for s in longterm_stocks[:3]]
                    )
                    
                    if selected_stocks:
                        # Calculate portfolio metrics
                        selected_data = [s for s in longterm_stocks if s['symbol'] in selected_stocks]
                        
                        col1, col2, col3 = st.columns(3)
                        
                        with col1:
                            avg_quality = np.mean([s['quality_score'] for s in selected_data])
                            st.metric("Avg Quality Score", f"{avg_quality:.1f}/100")
                        
                        with col2:
                            avg_dividend = np.mean([s['dividend_yield'] for s in selected_data])
                            st.metric("Avg Dividend Yield", f"{avg_dividend:.1f}%")
                        
                        with col3:
                            avg_pe = np.mean([s['pe_ratio'] for s in selected_data])
                            st.metric("Avg P/E Ratio", f"{avg_pe:.1f}")
                        
                        st.success(f"**Portfolio Recommendation**: {len(selected_stocks)}-stock portfolio " +
                                  f"with estimated annual dividend: ${capital * avg_dividend/100:.0f}")
                
                else:
                    st.warning("No long-term picks found. Please try again later.")
    
    # Tab 4: Stock Analyzer
    with tab4:
        st.markdown('📈 Individual Stock Analyzer', unsafe_allow_html=True)
        
        col1, col2 = st.columns([1, 3])
        
        with col1:
            # Stock selector
            symbol = st.text_input("Enter Stock Symbol", "AAPL").upper()
            period = st.selectbox(
                "Time Period",
                ["1mo", "3mo", "6mo", "1y", "2y", "5y"],
                index=3
            )
            
            analysis_type = st.selectbox(
                "Analysis Type",
                ["Technical", "Fundamental", "Both"],
                index=2
            )
            
            if st.button("Analyze Stock", type="primary"):
                st.session_state.analyze_stock = True
                st.session_state.current_symbol = symbol
        
        with col2:
            if 'analyze_stock' in st.session_state and st.session_state.analyze_stock:
                with st.spinner(f"Analyzing {st.session_state.current_symbol}..."):
                    # Get stock data
                    ticker = yf.Ticker(st.session_state.current_symbol)
                    info = ticker.info
                    
                    # Display basic info
                    st.markdown(f"## {info.get('longName', st.session_state.current_symbol)}")
                    
                    col_a, col_b, col_c, col_d = st.columns(4)
                    
                    with col_a:
                        current_price = info.get('currentPrice', info.get('regularMarketPrice', 0))
                        st.metric("Current Price", f"${current_price:.2f}")
                    
                    with col_b:
                        day_change = info.get('regularMarketChange', 0)
                        day_change_pct = info.get('regularMarketChangePercent', 0)
                        st.metric("Daily Change", 
                                 f"${day_change:.2f}" if day_change else "N/A",
                                 f"{day_change_pct:.2f}%" if day_change_pct else None)
                    
                    with col_c:
                        market_cap = info.get('marketCap', 0)
                        st.metric("Market Cap", f"${market_cap/1e9:.1f}B")
                    
                    with col_d:
                        pe_ratio = info.get('trailingPE', info.get('forwardPE', 0))
                        st.metric("P/E Ratio", f"{pe_ratio:.1f}")
                    
                    # Show chart
                    chart = analyzer.create_stock_chart(
                        st.session_state.current_symbol, 
                        period=period,
                        interval='1d' if period in ['1y', '2y', '5y'] else '1h'
                    )
                    
                    if chart:
                        st.plotly_chart(chart, use_container_width=True)
                    
                    # Fundamental Analysis
                    if analysis_type in ["Fundamental", "Both"]:
                        st.markdown("### 📊 Fundamental Analysis")
                        
                        fundamental_cols = st.columns(3)
                        
                        with fundamental_cols[0]:
                            st.markdown("**Valuation**")
                            valuation_metrics = {
                                "P/E Ratio": info.get('trailingPE'),
                                "Forward P/E": info.get('forwardPE'),
                                "P/B Ratio": info.get('priceToBook'),
                                "P/S Ratio": info.get('priceToSalesTrailing12Months'),
                                "PEG Ratio": info.get('pegRatio')
                            }
                            
                            for metric, value in valuation_metrics.items():
                                if value:
                                    st.markdown(f"{metric}: **{value:.2f}**")
                        
                        with fundamental_cols[1]:
                            st.markdown("**Profitability**")
                            profitability_metrics = {
                                "Profit Margin": info.get('profitMargins'),
                                "Operating Margin": info.get('operatingMargins'),
                                "ROE": info.get('returnOnEquity'),
                                "ROA": info.get('returnOnAssets')
                            }
                            
                            for metric, value in profitability_metrics.items():
                                if value:
                                    st.markdown(f"{metric}: **{value*100:.1f}%**")
                        
                        with fundamental_cols[2]:
                            st.markdown("**Growth**")
                            growth_metrics = {
                                "Revenue Growth": info.get('revenueGrowth'),
                                "EPS Growth": info.get('earningsGrowth'),
                                "Dividend Yield": info.get('dividendYield')
                            }
                            
                            for metric, value in growth_metrics.items():
                                if value:
                                    st.markdown(f"{metric}: **{value*100:.1f}%**")
                    
                    # Trading Recommendations
                    st.markdown("### 🎯 Trading Recommendations")
                    
                    try:
                        # Get historical data for decision making
                        hist = ticker.history(period='3mo')
                        if not hist.empty:
                            hist = analyzer.calculate_technical_indicators(hist)
                            
                            current_rsi = hist['RSI'].iloc[-1] if 'RSI' in hist.columns else 50
                            price_vs_sma20 = ((hist['Close'].iloc[-1] / hist['SMA_20'].iloc[-1]) - 1) * 100
                            
                            col_rec1, col_rec2, col_rec3 = st.columns(3)
                            
                            with col_rec1:
                                if current_rsi < 30:
                                    st.error("**Oversold** - Potential buying opportunity")
                                elif current_rsi > 70:
                                    st.warning("**Overbought** - Consider taking profits")
                                else:
                                    st.success("**Neutral** - No extreme signals")
                                st.markdown(f"RSI: {current_rsi:.1f}")
                            
                            with col_rec2:
                                if price_vs_sma20 > 5:
                                    st.success("**Strong Uptrend** - Above 20-day SMA")
                                elif price_vs_sma20 < -5:
                                    st.error("**Strong Downtrend** - Below 20-day SMA")
                                else:
                                    st.info("**Consolidating** - Near 20-day SMA")
                                st.markdown(f"vs SMA20: {price_vs_sma20:.1f}%")
                            
                            with col_rec3:
                                # Simple recommendation
                                if current_rsi < 40 and price_vs_sma20 < 0:
                                    recommendation = "Consider buying on weakness"
                                elif current_rsi > 60 and price_vs_sma20 > 0:
                                    recommendation = "Consider taking profits"
                                else:
                                    recommendation = "Hold or wait for better entry"
                                
                                st.markdown(f"**Action**: {recommendation}")
                    except:
                        st.info("Insufficient data for technical recommendations")

    # Tab 5: Risk Management
    with tab5:
        st.markdown('🛡️ Risk Management Tools', unsafe_allow_html=True)
        
        col1, col2 = st.columns(2)
        
        with col1:
            st.markdown('', unsafe_allow_html=True)
            st.markdown("### 📊 Position Sizing Calculator")
            
            entry_price = st.number_input("Entry Price ($)", min_value=0.01, value=100.0, step=0.01)
            stop_loss = st.number_input("Stop Loss Price ($)", min_value=0.01, value=95.0, step=0.01)
            risk_amount = st.number_input("Amount to Risk ($)", min_value=1.0, value=100.0, step=10.0)
            
            if entry_price > stop_loss:
                risk_per_share = entry_price - stop_loss
                shares = risk_amount / risk_per_share if risk_per_share > 0 else 0
                position_value = shares * entry_price
                
                st.markdown(f"**Position Size**: {shares:.0f} shares")
                st.markdown(f"**Position Value**: ${position_value:.2f}")
                st.markdown(f"**Risk per Share**: ${risk_per_share:.2f}")
                st.markdown(f"**Stop Loss %**: {(risk_per_share/entry_price)*100:.1f}%")
            else:
                st.warning("Entry price must be greater than stop loss")
            
            st.markdown('', unsafe_allow_html=True)
        
        with col2:
            st.markdown('', unsafe_allow_html=True)
            st.markdown("### 🎯 Risk-Reward Calculator")
            
            entry_rr = st.number_input("Entry Price ($)", min_value=0.01, value=100.0, step=0.01, key="rr_entry")
            stop_loss_rr = st.number_input("Stop Loss ($)", min_value=0.01, value=95.0, step=0.01, key="rr_stop")
            take_profit = st.number_input("Take Profit ($)", min_value=0.01, value=110.0, step=0.01)
            
            if entry_rr > stop_loss_rr and take_profit > entry_rr:
                risk = entry_rr - stop_loss_rr
                reward = take_profit - entry_rr
                rr_ratio = reward / risk if risk > 0 else 0
                
                st.markdown(f"**Risk**: ${risk:.2f} ({(risk/entry_rr)*100:.1f}%)")
                st.markdown(f"**Reward**: ${reward:.2f} ({(reward/entry_rr)*100:.1f}%)")
                st.markdown(f"**Risk-Reward Ratio**: 1:{rr_ratio:.1f}")
                
                if rr_ratio >= 2:
                    st.success("✓ Good risk-reward ratio")
                elif rr_ratio >= 1:
                    st.info("✓ Acceptable risk-reward ratio")
                else:
                    st.warning("✗ Poor risk-reward ratio")
            else:
                st.warning("Take profit must be greater than entry price")
            
            st.markdown('', unsafe_allow_html=True)
        
        # Portfolio Risk Analysis
        st.markdown("### 📈 Portfolio Risk Analysis")
        
        portfolio_size = st.number_input("Portfolio Size ($)", min_value=1000, value=capital, step=1000)
        
        # Simulated portfolio allocation
        allocations = {
            "Stock Type": ["Large Cap", "Small Cap", "International", "Bonds", "Cash"],
            "Conservative": [30, 10, 20, 30, 10],
            "Moderate": [40, 15, 20, 20, 5],
            "Aggressive": [50, 20, 20, 5, 5]
        }
        
        df_allocation = pd.DataFrame(allocations)
        
        # Show recommended allocation based on risk profile
        st.markdown(f"**Recommended Allocation for {risk_profile} Profile:**")
        selected_allocation = df_allocation[["Stock Type", risk_profile]]
        
        # Create a pie chart
        fig = go.Figure(data=[go.Pie(
            labels=selected_allocation["Stock Type"],
            values=selected_allocation[risk_profile],
            hole=.3
        )])
        
        fig.update_layout(
            title="Portfolio Allocation",
            height=400
        )
        
        st.plotly_chart(fig, use_container_width=True)
        
        # Drawdown Calculator
        st.markdown("### 📉 Maximum Drawdown Calculator")
        
        initial_portfolio = st.number_input("Initial Portfolio Value ($)", min_value=1000, value=10000, step=1000)
        worst_case = st.slider("Worst Case Decline (%)", min_value=5, max_value=50, value=20, step=5)
        
        drawdown_amount = initial_portfolio * (worst_case / 100)
        remaining_amount = initial_portfolio - drawdown_amount
        
        col_a, col_b = st.columns(2)
        
        with col_a:
            st.metric("Maximum Drawdown", f"${drawdown_amount:.0f}")
        
        with col_b:
            st.metric("Remaining Value", f"${remaining_amount:.0f}")
        
        st.warning(f"⚠️ In a {worst_case}% market decline, your ${initial_portfolio:,} portfolio would drop to ${remaining_amount:,}")
        
        # Risk Management Rules
        st.markdown("### 📋 Essential Risk Management Rules")
        
        rules = [
            "🔴 **Never risk more than 2% of your capital on a single trade**",
            "🟡 **Always use stop-loss orders**",
            "🟢 **Keep risk-reward ratio at least 1:2**",
            "🔵 **Diversify across sectors and asset classes**",
            "🟣 **Rebalance portfolio quarterly**",
            "🟠 **Have an exit strategy before entering any position**"
        ]
        
        for rule in rules:
            st.markdown(rule)

# ==================== RUN APPLICATION ====================
if __name__ == "__main__":
    main()
