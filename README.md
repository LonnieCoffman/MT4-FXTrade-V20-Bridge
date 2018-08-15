# MT4 to FXTrade V20 Bridge

This Python script to act as a bridge from MT4 to Oanda FXtrade. I use this to allow my MT4 expert advisors to send trades to Oanda's FXtrade V20 API. Using this I can place trades as small as 1 unit.

The MQL script writes files to the files/FXtrade folder and this Python script reads them and manipulates orders via the Oanda API. Reading and writing files to communicate orders was the most consistent method that I found to get data from MT4 to Python. It is also unlikely that a future MT4 update will disable file usage.

### Requirements
---
[oandapyV20](https://github.com/hootnot/oanda-api-v20)
requests
six

### Configuration
---
Create a folder named FXtrade within the /MQL4/Files directory of your MT4 installation.  Edit the details within the static.py file using your Oanda connection details and the directory path to the FXtrade folder you just created.

### Trade Functions
---
I do not use pending orders but rather allow my experts to monitor when to open and close trades, so the functionality is limited to the following.

+ Open market order with/without stoploss and target
+ Add on to existing trade
+ Fully or partially close trade


```
//================================================//
// FXtrade Bridge Functions                       //
//================================================//
// create order file
bool OpenMarketOrder(string instrument, string side, int units, double stop=0.0, double target=0.0){
   int filehandle;
   bool order;
   string pair = instrument;
   StringReplace(pair,"_","");
   
   string command = "openmarket-"+instrument+"-"+side+"-"+IntegerToString(units)+"-"+DoubleToStr(stop,int(MarketInfo(pair,MODE_DIGITS)))+"-"+DoubleToStr(target,int(MarketInfo(pair,MODE_DIGITS)));

   LockDirectory();
   filehandle=FileOpen("FXtrade\\"+command,FILE_WRITE|FILE_TXT);
   if(filehandle!=INVALID_HANDLE){
      FileClose(filehandle);
      order = True;
   } else order = False;
   UnlockDirectory();
   Sleep(5000);
   return order;
}

// create close position file
bool ClosePosition(string instrument, string side, int units=0){
   int filehandle;
   filehandle=FileOpen("FXtrade\\close-"+instrument+"-"+side+"-"+IntegerToString(units),FILE_WRITE|FILE_TXT);
   if(filehandle!=INVALID_HANDLE){
      FileClose(filehandle);
      return True;
   } else return False;
}

// lock directory so python does not access files
bool LockDirectory(){
   int filehandle;
   filehandle=FileOpen("FXtrade\\MT4-Locked",FILE_WRITE|FILE_TXT);
   if(filehandle!=INVALID_HANDLE){
      FileClose(filehandle);
      return True;
   } else return False;
}

// unlock directory so python can access files
bool UnlockDirectory(){
   int filehandle;
   filehandle=FileDelete("FXtrade\\MT4-Locked");
   if (filehandle == False) return False;
      else return True;
}
```