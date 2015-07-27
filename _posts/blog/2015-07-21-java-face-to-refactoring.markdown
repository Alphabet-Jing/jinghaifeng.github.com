# 遇到的重构问题

	public boolean sleepIn(boolean weekday, boolean vacation) {
    boolean sleepIn = false;
    
    if (vacation){
        sleepIn = true;
    }
    else{
        if (!weekday){
            sleepIn = true;
        }
    }
    return sleepIn;
	}

其实可重构成，逻辑更清晰。

	public boolean sleepIn(boolean weekday, boolean vacation) {
    	if (!weekday || vacation){
		return true;
		}
		return false;
	}
