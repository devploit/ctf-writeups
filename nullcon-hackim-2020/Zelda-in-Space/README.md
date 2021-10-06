### 0x01 Challenge:

**Description:**

<img src="https://i.imgur.com/O0zkfmM.png" width=300 height=400>

### 0x02 Write-up:

For this last challenge we are provided with a new game "Zelda in Space". If we check the methods in dnSpy as in the previous challenges, we see that it has an "AntiHack" that kicks out cheating players.

![Branching](https://i.imgur.com/zDGPH2P.png)

But what interests us in this challenge is to find the 6 buttons and THE ORDER TO PUSH. We see how inside RoomMove > OnTriggerEnter2D these 6 buttons are referred to, which are named by ingredient names.

![Branching](https://i.imgur.com/AgZSZjQ.png)

On the other hand we see the classes contained in the PlayerAttrs method. In OnGUI we have the execution of the Decrypt class if we reach the checkpoint called "final".

![Branching](https://i.imgur.com/e0AQAH5.png)

We also reviewed the Decrypt class to see how they carry out this process.

![Branching](https://i.imgur.com/R70OtAd.png)

Finally, it is important to note that our character starts with the checkpoint "0" hardcoded.

![Branching](https://i.imgur.com/D1KjLO2.png)

With all this data (and in consideration of that being 6 factorial there is a 720 possible results) we generated a small program that helps us to decrypt the flag returning us the order in which we have to add the ingredients.

```C#
using System;
using System.Collections.Generic;
using System.IO;
using System.Security.Cryptography;
using System.Text;
using System.Threading;
 
public class Program
{
    public List<string> cpoints;
    public string encrypted;
    public string f;
    public Random rng = new Random();  
   
    public void Main()
    {
        cpoints = new List<string>();
        cpoints.Add("pepper");
        cpoints.Add("salt");
        cpoints.Add("chilly");
        cpoints.Add("pickles");
        cpoints.Add("oregano");
        cpoints.Add("masala");
       
        encrypted = string.Empty;
        f="0";
        string text = "";
        while(f == "0" && cpoints.Count >5 )
        {
        text = "";
        StringBuilder stringBuilder = new StringBuilder();
       
        cpoints.Shuffle(); 
           
        foreach (string checkpoint in cpoints)
        {
            stringBuilder.Append(checkpoint);
        }
       
        text = stringBuilder.ToString();
        text = "0"+text+"final";
        try
        {
            encrypted = Decrypt("pI0gDg911A3Qcf++L3rvfkwIEkXsg4jq6pwOHMgG1VlpPuE9t4eljr4fQvXUa9bMJN4TL+DzQoj8aHTe1sNt+y5FND+gqn04OOltMhv/sms=", text);
            f="1";
        }
        catch (CryptographicException)
        {
            encrypted = "wrong key";
            f="0";
        }
       
        }
        if (!string.Equals(text, string.Empty))
        {
            Console.WriteLine(text);
        }
        if (!string.Equals(encrypted, string.Empty))
        {
            Console.WriteLine(encrypted);
        }
    }
   
    private string Decrypt(string cipherText, string key)
    {
        byte[] array = Convert.FromBase64String(cipherText);
        using (Aes aes = Aes.Create())
        {
            Rfc2898DeriveBytes rfc2898DeriveBytes = new Rfc2898DeriveBytes(key, new byte[13]
            {
                73,
                118,
                97,
                110,
                32,
                77,
                101,
                100,
                118,
                101,
                100,
                101,
                118
            });
            aes.Key = rfc2898DeriveBytes.GetBytes(32);
            aes.IV = rfc2898DeriveBytes.GetBytes(16);
            using (MemoryStream memoryStream = new MemoryStream())
            {
                using (CryptoStream cryptoStream = new CryptoStream(memoryStream, aes.CreateDecryptor(), CryptoStreamMode.Write))
                {
                    cryptoStream.Write(array, 0, array.Length);
                    cryptoStream.Close();
                }
                cipherText = Encoding.Unicode.GetString(memoryStream.ToArray());
                return cipherText;
            }
        }
    }
}
 
static class MyExtensions
  {
    public static void Shuffle<T>(this IList<T> list)
    {
      int n = list.Count;
      while (n > 1)
      {
        n--;
        int k = ThreadSafeRandom.ThisThreadsRandom.Next(n + 1);
        T value = list[k];
        list[k] = list[n];
        list[n] = value;
      }
    }
  }
 
 public static class ThreadSafeRandom
  {
      [ThreadStatic] private static Random Local;
 
      public static Random ThisThreadsRandom
      {
          get { return Local ?? (Local = new Random(unchecked(Environment.TickCount * 31 + Thread.CurrentThread.ManagedThreadId))); }
      }
  }
```

By executing it, it gives us back...

![Branching](https://i.imgur.com/FVKEBO4.png)

If we wanted to check directly on the game, we'd add all the ingredients and go to the final platform of the game to have the "final" string added and check that everything is correct.

![Branching](https://i.imgur.com/PT9iCJ9.png)

![Branching](https://i.imgur.com/hc2g1oE.png)

`Flag: hackim20{z3lda_s0lved_the_sp4ce_puzzl3}`
