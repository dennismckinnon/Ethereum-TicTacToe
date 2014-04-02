Shamir's Secret Sharing Algorithm for Bitcoin Private Keys
--------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------
Written by: Dennis McKinnon - dennis.r.mckinnon@gmail.com
Polynomial class by: Andrew Brown


If you like this code feel free to send some BTC to 1G4KSVirqUnNG2DNThuq3JhuUqcAXpL2hW

--------------------------------------------------------------------------------------------------
This is an implementation of Shamir's Secret Sharing algorithm
http://en.wikipedia.org/wiki/Shamir's_Secret_Sharing Specifically for Bitcoin Private Keys
It works in the Field of integers mod 59

This code is provided for free unrestricted use. You may copy any parts delete any parts even this
README if you so choose. I do not require credit or hold any ownership over any of this code.


Polynomial Class: Allows for manipulation of polynomial like objects (very useful)
intmod class: Implements integers mod base. NOTE base MUST be set using set_base(b) before its used

ShSS.py: Implementation of Shamir's Secret Sharing (See Below for more)
shtest.py: Testing suite proving Algorithm is working as hoped

--------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------
Command line usage:
python ShSS.py [-p "password"]{-s "n" "k" "Secret"/-sf "n" "k" "File"/-r "Shares"/ -rf "File"}

-p "password" - optional argument, followed immediately by the password to use
-s "n" "k" "Secret" - Specifies that "Secret" is to be split with n shares produced a minimum
                      of k being needed to successfully reconstruct the secret
-sf "n" "k" "File" - Same as -s but the secret is found in file "File"
-r "Shares" - Specifies that the "Shares" entered after -r (space separated) should be used to
              attempt recovery of the secret
-rf "File" - Same as -r except the shares are stored in the file "File" (One per line)
   
only one of (-s,-sf,-r,-rf) may be used. -p may be used with all arguments.
main() handles command line input... Not fully idiot-proof

-------------------------------------------------------------------------------------------------
def split(n, k, secret{, password}):
n - Number of shares to produce, n>=k. if n<k, n is set to k
k - Threshold number of shares to reconstruct the secret
secret - The secret to be split. In this implementation this is a string of characters in
        base 58. This is so it works well with Bitcoin private addresses
password (optional) - ASCII string password which will be hashed and the result added to
                     the secret (one-time pad)
                          
Method implements Shamirs Secret sharing on a Field of integers mod 59. A sequence of
independent random polynomials are constructed for each character c of the secret such that
for each polynomial p, p(0)=c (more or less see b58conv and r58conv).
See http://en.wikipedia.org/wiki/Shamir's_Secret_Sharing for a detailed description of how
Shamir's Secret Sharing works

------------------------------------------------------------------------------------------------
def recover(shares[]{, password}):
shares[]- List of the secret shares
password (optional) - ASCII string password which will be hashed and the result added to
                      the secret (one-time pad) 
  
This method should reconstruct a secret from a certain number of shares
It will always assume enough shares are provided to reconstruct the secret
(By assuming the degree of the polynomial is one less then the number of shares)
   
What this means is if the threshold of reconstruction was k, and k'<k was provided
the output will not be guaranteed to be the secret
   
The probability that the k'<k shares will reconstruct the polynomial should be be
the probability that k'-1 shares determine the last which yeilds a constant 1/59
probability per character for a total probability that the secret is randomly revealed
as (1/59)^len(message). However it should be impossible for the attacker to know for sure
he has the secret (except by testing it)
  
Since the Field we are working with in finite, to brute-force this if you have k'=k-1 shares
you need to guess each of the 59 values for each character in the message, this is equivalent
to brute-forcing the original secret

-------------------------------------------------------------------------------------------------   
def b58conv(C):
C - Character in base 58 to be converted to an integer between 0 and 58
   
This is a helper function which performs the conversion of base 58 characters (plus 0) into
their integer equivalents
    
-------------------------------------------------------------------------------------------------
def r58conv(N):
N - Integer between 0 and 58 to be converted to a character in base 58
    
This performs the reverse operation to b58conv in that it will take integers mod 59 to their
equivalent in base 58 (plus 0)

-------------------------------------------------------------------------------------------------
def genpad(len,pword):
len - length of pad to create using the password
pword - Password to generate the pad from

This function is designed to create a re-creatable one time pad from a provided password. It was
included to remove a bias that was in the previous method to do this which could lead to breaking
the key if you had the same key under several encryptions.

------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------
USAGE EXAMPLES:

Case: You want to split the private key so any two pieces out of 3 can reconstruct the secret
python ShSS -s 3 2 "Secret"

Case: Same as before but in order for the secret to be obtained two shares are needed AND a password
python ShSS -p "Password" -s 3 2 "Secret"

Case: You just want to encrypt your key with a password
python ShSS -p "Password" -s 1 1 "Secret"

Case: You want three pieces to reveal the secret without password OR one share with a password
Combine a couple previous scenarios namely
python ShSS -p "Password" -s 1 1 "Secret"
AND
python ShSS -s 3 3 "Secret"

------------------------------------------------------------------------------------------------




