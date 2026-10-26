Backtrader, NautilusTrader, QuantConnect LEAN and Reamer Research: the same data, the same strategies, one 64-core AMD EPYC 9575F.

I build Reamer Research, so you should not take my word for the result. That is how I set up the study. I measured how wall time and memory grow in three directions: with longer data, with more tickers, and with more copies running at once.

Before I saw any result, I wrote down four outcomes that would count against Reamer Research. Two of them occurred:
→ Above about 270,000 bars, Backtrader used less memory than our Python path (209 MB against 309 MB at 500,000 bars).
→ At 64 parallel copies, our C++ path had the lowest parallel efficiency of the four engines, 0.53 against 0.63 to 0.72.

The run also found a bug in our own Python binding: a fixed-size trade buffer that failed on the longest runs. The fix shipped in 4.6.1 the next day, and the paper says which results include it.

What else the run showed:
→ One 500,000-bar backtest took 0.36 s on our C++ path and 3.4 s on our Python path. The other engines took 13.6 to 22.1 s.
→ At 64 copies, the C++ path completed 332,249 backtests an hour. Patched NautilusTrader completed 11,242 and Backtrader 11,125.
→ Stock NautilusTrader 1.231.0 records PnL at a cost that grows with the square of the trade count. This was already reported upstream (#4434), and the maintainers say 2.0 removes it. The paper gives both the stock and the patched results.
→ Every completed run of every engine repeated its result.

Speed is not the whole choice. LEAN has the broadest asset and order model and a hosted path to live trading. NautilusTrader has built-in live adapters on one code path. Backtrader is pure Python and easy to change. Three of the four engines cost nothing to license. The paper compares these points too, with sources.

The runners and raw results are public. If you think an engine deserved a better configuration, run it and tell me what you get.

Paper: https://doi.org/10.6084/m9.figshare.34787745
Data and runners: https://doi.org/10.6084/m9.figshare.34785729

#backtesting #algotrading #quantitativefinance #QuantConnect #NautilusTrader #Backtrader #reproducibility
