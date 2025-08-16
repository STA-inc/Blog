```java
package Main;
public class Main {
	public static void main(String[] args) {
		System.out.println(getimei15("86126303166563"));
	}
	public static String getimei15(String imei){
        if (imei.length() == 14) {
            char[] imeiChar=imei.toCharArray();  
            int resultInt=0;  
            for (int i = 0; i < imeiChar.length; i++) {  
                int a=Integer.parseInt(String.valueOf(imeiChar[i]));  
                i++;  
                final int temp=Integer.parseInt(String.valueOf(imeiChar[i]))*2;  
                final int b=temp<10?temp:temp-9;  
                resultInt+=a+b;  
            }  
            resultInt%=10;  
            resultInt=resultInt==0?0:10-resultInt;  
            return imei+resultInt + "";
        }else{
            return "";
        }
    }
}
```
