//+------------------------------------------------------------------+
//| Expert Advisor pour XAU/USD avec RSI et SMA                      |
//| Ce robot prend des positions d'achat et de vente en fonction des  |
//| signaux RSI et SMA sur le marché XAU/USD                          |
//+------------------------------------------------------------------+
#property strict

// Paramètres de l'utilisateur
input int smaPeriod = 50;               // Période de la Moyenne Mobile Simple (SMA)
input int rsiPeriod = 14;               // Période du RSI
input double overbought = 70;           // Seuil de surachat pour le RSI
input double oversold = 30;             // Seuil de survente pour le RSI
input double lotSize = 0.1;             // Taille du lot pour chaque ordre
input int slippage = 3;                // Slippage autorisé
input int stopLoss = 100;              // Stop Loss en pips
input int takeProfit = 200;            // Take Profit en pips
input int magicNumber = 123456;        // Numéro magique pour identifier les ordres

// Déclaration des variables pour les indicateurs
double smaCurrent, smaPrevious;        // Valeurs actuelles et précédentes de la SMA
double rsiCurrent, rsiPrevious;        // Valeurs actuelles et précédentes du RSI

// Fonction d'initialisation
int OnInit()
  {
   // Initialisation de l'EA
   Print("Expert Advisor pour XAU/USD initialisé !");
   return(INIT_SUCCEEDED);
  }

// Fonction qui s'exécute à chaque tick
void OnTick()
  {
   // Calculer la valeur actuelle et précédente de la SMA
   smaCurrent = iMA(Symbol(), PERIOD_H1, smaPeriod, 0, MODE_SMA, PRICE_CLOSE, 0);
   smaPrevious = iMA(Symbol(), PERIOD_H1, smaPeriod, 0, MODE_SMA, PRICE_CLOSE, 1);
   
   // Calculer la valeur actuelle et précédente du RSI
   rsiCurrent = iRSI(Symbol(), PERIOD_H1, rsiPeriod, PRICE_CLOSE, 0);
   rsiPrevious = iRSI(Symbol(), PERIOD_H1, rsiPeriod, PRICE_CLOSE, 1);

   // Vérifier les conditions d'achat
   if (rsiCurrent < oversold && Close[0] > smaCurrent && Close[1] <= smaPrevious)
     {
      // Conditions d'achat : RSI sous 30 (survente) et prix au-dessus de la SMA
      if (PositionSelect(Symbol()) == false || PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL)
        {
         // Fermer une position de vente existante si elle existe
         CloseSellPosition();
         
         // Ouvrir une position d'achat
         OpenBuyPosition();
        }
     }

   // Vérifier les conditions de vente
   if (rsiCurrent > overbought && Close[0] < smaCurrent && Close[1] >= smaPrevious)
     {
      // Conditions de vente : RSI au-dessus de 70 (surachat) et prix en dessous de la SMA
      if (PositionSelect(Symbol()) == false || PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY)
        {
         // Fermer une position d'achat existante si elle existe
         CloseBuyPosition();
         
         // Ouvrir une position de vente
         OpenSellPosition();
        }
     }
  }

// Fonction pour ouvrir une position d'achat
void OpenBuyPosition()
  {
   double price = Ask;
   double sl = price - stopLoss * Point;  // Calcul du Stop Loss
   double tp = price + takeProfit * Point;  // Calcul du Take Profit
   
   // Ouvrir une position d'achat
   int ticket = OrderSend(Symbol(), OP_BUY, lotSize, price, slippage, sl, tp, "Buy Order", magicNumber, clrGreen);
   if(ticket < 0)
     {
      Print("Erreur lors de l'ouverture d'une position d'achat : ", GetLastError());
     }
  }

// Fonction pour ouvrir une position de vente
void OpenSellPosition()
  {
   double price = Bid;
   double sl = price + stopLoss * Point;  // Calcul du Stop Loss
   double tp = price - takeProfit * Point;  // Calcul du Take Profit
   
   // Ouvrir une position de vente
   int ticket = OrderSend(Symbol(), OP_SELL, lotSize, price, slippage, sl, tp, "Sell Order", magicNumber, clrRed);
   if(ticket < 0)
     {
      Print("Erreur lors de l'ouverture d'une position de vente : ", GetLastError());
     }
  }

// Fonction pour fermer une position d'achat
void CloseBuyPosition()
  {
   int ticket = PositionGetInteger(POSITION_TICKET);
   if (ticket > 0 && PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_BUY)
     {
      if (!OrderClose(ticket, PositionGetDouble(POSITION_VOLUME), Bid, slippage, clrGreen))
        {
         Print("Erreur lors de la fermeture de la position d'achat : ", GetLastError());
        }
     }
  }

// Fonction pour fermer une position de vente
void CloseSellPosition()
  {
   int ticket = PositionGetInteger(POSITION_TICKET);
   if (ticket > 0 && PositionGetInteger(POSITION_TYPE) == POSITION_TYPE_SELL)
     {
      if (!OrderClose(ticket, PositionGetDouble(POSITION_VOLUME), Ask, slippage, clrRed))
        {
         Print("Erreur lors de la fermeture de la position de vente : ", GetLastError());
        }
     }
  }
