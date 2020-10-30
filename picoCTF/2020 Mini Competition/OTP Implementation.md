## Blog
https://frc6.com/index.php/archives/39/

## Source and Problem
https://play.picoctf.org/events/3/challenges/challenge/92

[otp.zip][1]

## Tool
IDA Pro

## Conditions of correct key
Disassembly the file otp. Read the assembly code overall. 

The following code showed that key should satisfy 2 condition:

 - length ([rbp+var_E8]) equal 100
 - s1 equal s2 which is "mngjlepdcbcmjmmjipmmegfkjbicaemoemkkpjgnhgomlknmoepmfbcoffikhplmadmganmlojndmfahbhaancamdhfdkiancdjf"

In Disassembly,

    .text:0000557E7FE4E99D                 cmp     [rbp+var_E8], 64h ; 'd'
    .text:0000557E7FE4E9A4                 jnz     short loc_557E7FE4E9C6 ; if ( i == 100
    .text:0000557E7FE4E9A4                                         ; line 35
    .text:0000557E7FE4E9A6                 mov     eax, [rbp+var_E8] ;       && !strncmp(
    .text:0000557E7FE4E9A6                                         ;             s1,
    .text:0000557E7FE4E9A6                                         ;             "mngjlepdcbcmjmmjipmmegfkjbicaemoemkkpjgnhgomlknmoepmfbcoffikhplmadmganmlojndmfahbhaancamdhfdkiancdjf",
    .text:0000557E7FE4E9A6                                         ;             0x64uLL) )
    .text:0000557E7FE4E9A6                                         ; line 36-39
    .text:0000557E7FE4E9AC                 movsxd  rdx, eax        ; n = i = [rbp+var_E8]
    .text:0000557E7FE4E9AF                 lea     rax, [rbp+s1]
    .text:0000557E7FE4E9B3                 lea     rsi, s2         ; "mngjlepdcbcmjmmjipmmegfkjbicaemoemkkpjg"...
    .text:0000557E7FE4E9BA                 mov     rdi, rax        ; s1
    .text:0000557E7FE4E9BD                 call    _strncmp        ; test if s1 equal s2
    .text:0000557E7FE4E9C2                 test    eax, eax
    .text:0000557E7FE4E9C4                 jz      short loc_557E7FE4E9D9 ; if i = 100 and s1 = s2，then key is correct

In Pseudocode,

    if ( i == 100
      && !strncmp(
            s1,
            "mngjlepdcbcmjmmjipmmegfkjbicaemoemkkpjgnhgomlknmoepmfbcoffikhplmadmganmlojndmfahbhaancamdhfdkiancdjf",
            0x64uLL) )

## Analysis algorithm (input: dest; ouput: s1)

    Analysis the disassembly code from 0000557E7FE4E89E to 0000557E7FE4E99B

In Disassembly,

    .text:0000557E7FE4E89E ; ---------------------------------------------------------------------------
    .text:0000557E7FE4E89E
    .text:0000557E7FE4E89E loc_557E7FE4E89E:                       ; CODE XREF: main+14A↓j
    .text:0000557E7FE4E89E                 cmp     [rbp+var_E8], 0 ; if ( i )
    .text:0000557E7FE4E89E                                         ; line 21
    .text:0000557E7FE4E8A5                 jnz     short loc_557E7FE4E8E4
    .text:0000557E7FE4E8A7                 mov     eax, [rbp+var_E8] ; when i equal 0
    .text:0000557E7FE4E8A7                                         ; line 30
    .text:0000557E7FE4E8AD                 cdqe
    .text:0000557E7FE4E8AF                 movzx   eax, [rbp+rax+dest]
    .text:0000557E7FE4E8B7                 movsx   eax, al
    .text:0000557E7FE4E8BA                 mov     edi, eax
    .text:0000557E7FE4E8BC                 call    jumble
    .text:0000557E7FE4E8C1                 mov     edx, eax
    .text:0000557E7FE4E8C3                 mov     eax, edx
    .text:0000557E7FE4E8C5                 sar     al, 7
    .text:0000557E7FE4E8C8                 shr     al, 4
    .text:0000557E7FE4E8CB                 add     edx, eax
    .text:0000557E7FE4E8CD                 and     edx, 0Fh
    .text:0000557E7FE4E8D0                 sub     edx, eax
    .text:0000557E7FE4E8D2                 mov     eax, edx
    .text:0000557E7FE4E8D4                 mov     edx, eax
    .text:0000557E7FE4E8D6                 mov     eax, [rbp+var_E8]
    .text:0000557E7FE4E8DC                 cdqe
    .text:0000557E7FE4E8DE                 mov     [rbp+rax+s1], dl
    .text:0000557E7FE4E8E2                 jmp     short loc_557E7FE4E935
    .text:0000557E7FE4E8E4 ; ---------------------------------------------------------------------------
    .text:0000557E7FE4E8E4
    .text:0000557E7FE4E8E4 loc_557E7FE4E8E4:                       ; CODE XREF: main+97↑j
    .text:0000557E7FE4E8E4                 mov     eax, [rbp+var_E8] ; when i not equal 0
    .text:0000557E7FE4E8E4                                         ; line 23-26
    .text:0000557E7FE4E8EA                 cdqe
    .text:0000557E7FE4E8EC                 movzx   eax, [rbp+rax+dest]
    .text:0000557E7FE4E8F4                 movsx   eax, al
    .text:0000557E7FE4E8F7                 mov     edi, eax        ; edi = dest[i]
    .text:0000557E7FE4E8F9                 call    jumble          ; line 23
    .text:0000557E7FE4E8FE                 movsx   edx, al
    .text:0000557E7FE4E901                 mov     eax, [rbp+var_E8]
    .text:0000557E7FE4E907                 sub     eax, 1
    .text:0000557E7FE4E90A                 cdqe
    .text:0000557E7FE4E90C                 movzx   eax, [rbp+rax+s1]
    .text:0000557E7FE4E911                 movsx   eax, al         ; eax = s1[i-1]
    .text:0000557E7FE4E914                 add     edx, eax        ; edx = v5
    .text:0000557E7FE4E914                                         ; line 24
    .text:0000557E7FE4E916                 mov     eax, edx
    .text:0000557E7FE4E918                 sar     eax, 1Fh
    .text:0000557E7FE4E91B                 shr     eax, 1Ch        ; line 25
    .text:0000557E7FE4E91E                 add     edx, eax
    .text:0000557E7FE4E920                 and     edx, 0Fh
    .text:0000557E7FE4E923                 sub     edx, eax        ; line 26
    .text:0000557E7FE4E925                 mov     eax, edx
    .text:0000557E7FE4E927                 mov     edx, eax
    .text:0000557E7FE4E929                 mov     eax, [rbp+var_E8]
    .text:0000557E7FE4E92F                 cdqe
    .text:0000557E7FE4E931                 mov     [rbp+rax+s1], dl ; [rbp+rax+s1] = s1[i]
    .text:0000557E7FE4E935
    .text:0000557E7FE4E935 loc_557E7FE4E935:                       ; CODE XREF: main+D4↑j
    .text:0000557E7FE4E935                 add     [rbp+var_E8], 1 ; ++i
    .text:0000557E7FE4E93C
    .text:0000557E7FE4E93C loc_557E7FE4E93C:                       ; CODE XREF: main+8B↑j
    .text:0000557E7FE4E93C                 mov     eax, [rbp+var_E8] ; for ( i = 0; (unsigned int)valid_char(dest[i]); ++i )
    .text:0000557E7FE4E93C                                         ; line 19-32
    .text:0000557E7FE4E942                 cdqe
    .text:0000557E7FE4E944                 movzx   eax, [rbp+rax+dest]
    .text:0000557E7FE4E94C                 movsx   eax, al
    .text:0000557E7FE4E94F                 mov     edi, eax        ; edi = dest[i]
    .text:0000557E7FE4E951                 call    valid_char      ; valid_char(dest[i])
    .text:0000557E7FE4E956                 test    eax, eax
    .text:0000557E7FE4E958                 jnz     loc_557E7FE4E89E
    .text:0000557E7FE4E95E                 mov     [rbp+var_E4], 0 ; j
    .text:0000557E7FE4E968                 jmp     short loc_557E7FE4E98F
    .text:0000557E7FE4E96A ; ---------------------------------------------------------------------------
    .text:0000557E7FE4E96A
    .text:0000557E7FE4E96A loc_557E7FE4E96A:                       ; CODE XREF: main+18D↓j
    .text:0000557E7FE4E96A                 mov     eax, [rbp+var_E4]
    .text:0000557E7FE4E970                 cdqe
    .text:0000557E7FE4E972                 movzx   eax, [rbp+rax+s1]
    .text:0000557E7FE4E977                 add     eax, 61h ; 'a'
    .text:0000557E7FE4E97A                 mov     edx, eax
    .text:0000557E7FE4E97C                 mov     eax, [rbp+var_E4]
    .text:0000557E7FE4E982                 cdqe
    .text:0000557E7FE4E984                 mov     [rbp+rax+s1], dl
    .text:0000557E7FE4E988                 add     [rbp+var_E4], 1
    .text:0000557E7FE4E98F
    .text:0000557E7FE4E98F loc_557E7FE4E98F:                       ; CODE XREF: main+15A↑j
    .text:0000557E7FE4E98F                 mov     eax, [rbp+var_E4] ; for ( j = 0; j < i; ++j )
    .text:0000557E7FE4E98F                                         ; line 33-34
    .text:0000557E7FE4E995                 cmp     eax, [rbp+var_E8]
    .text:0000557E7FE4E99B                 jl      short loc_557E7FE4E96A

In Pseudocode,

    for ( i = 0; (unsigned int)valid_char(dest[i]); ++i )// test if dest[i] is 1-9 or a-f
    {
      if ( i )
      {
        v4 = jumble(dest[i]);
        v5 = s1[i - 1] + v4;
        v6 = (unsigned int)((s1[i - 1] + v4) >> 31) >> 28;
        s1[i] = ((v6 + v5) & 0xF) - v6;
      }
      else
      {
        s1[0] = (char)jumble(dest[0]) % 16;
      }
    }
    for ( j = 0; j < i; ++j )
      s1[j] += 97;

This is an algorithm in which input is dest and output is s1. 

## Find Key [Java Code (Reversely：s2 => s1 => dest)]
Through exhaustive idea, write the java programming reversely: find dest from the known s1 (which should equal s2)

#### Find dest[0]
s2[0] = "m" = 109
s1[0] = s2[0] - 97 = 12

Enumerate all the possibilities of s1[0] and find the case where s1[0] is equal to 12

    public static void main(String[] args)
    {
    	System.out.println("dest[0]|s1[0]"); 
    	int[] a1s = {48,49,50,51,52,53,54,55,56,57,97,98,99,100,101,102};
    	for(int i = 0; i < 16; i++)
    	{
        	System.out.println(a1s[i] + "|" +jumble(a1s[i]) % 16);   		
    	}
    }
	
    public static int jumble(int a1)
    {
    	int v2 = a1;
    	if ( a1 > 96 )                                // a-f
    	    v2 = a1 + 9;                                // j-o
    	int v3 = 2 * (v2 % 16);
    	if ( (char)v3 > 15 )
    	    ++v3;
    	return v3;
    }

Output:
    dest[0]|s1[0]
    48|0
    49|2
    50|4
    51|6
    52|8
    53|10
    54|12
    55|14
    56|1
    57|3
    97|5
    98|7
    99|9
    100|11
    101|13
    102|15

Conclusion：When dest[0] equal 54 (char “6”), s1[0] equal 12.

#### Find dest[1]
s2[1] = "n" = 110

Enumerate all the possibilities of s1[1] and find the case where s1[1] is equal to 110, and find dest[1]

    public static void main(String[] args)
    {
    	s1[0] = 12;
    	System.out.println("dest[1]|s1[1]"); 
    	int[] a1s = {48,49,50,51,52,53,54,55,56,57,97,98,99,100,101,102};
    	for(int i = 0; i < 16; i++)
    	{
    		getMS1IndexNotEqual0(a1s[i],1);
    		System.out.println(a1s[i] + "|" +s1[1]);   		
    	}
    }
    
    static int[] s1 = new int[100];
    
    //find dest[not equal 0]
    //mD = dest[i]
    public static int getMS1IndexNotEqual0(int mD, int i)
    {
    	if(i == 0)
    	{
    		return -1;
    	}
    	int v4 = jumble(mD);
    	int v5 = s1[i - 1] + v4;
    	int v6 = (v5 >> 31) >>> 28;
    	s1[i] = ((v6 + v5) & 0xF) - v6;
    	s1[i] += 97;
    	return s1[i];
    }
	
    public static int jumble(int a1)
    {
    	int v2 = a1;
    	if ( a1 > 96 )                                // a-f
    	    v2 = a1 + 9;                                // j-o
    	int v3 = 2 * (v2 % 16);
    	if ( (char)v3 > 15 )
    	    ++v3;
    	return v3;
    }

Output:

    dest[1]|s1[1]
    48|109
    49|111
    50|97
    51|99
    52|101
    53|103
    54|105
    55|107
    56|110
    57|112
    97|98
    98|100
    99|102
    100|104
    101|106
    102|108

Conclusion：When dest[1] equal 56 (char “8”), s1[1] equal 110.

#### Find dest[1-100] by programming automaticly
    public static void auto()
    {
    	s1[0] = 12;
    	String s2 = "mngjlepdcbcmjmmjipmmegfkjbicaemoemkkpjgnhgomlknmoepmfbcoffikhplmadmganmlojndmfahbhaancamdhfdkiancdjf";
    	char[] a1s = {48,49,50,51,52,53,54,55,56,57,97,98,99,100,101,102};
    	boolean hasResult = false;
    	for(int i2 = 1; i2 < 100; i2++)
    	{
    		hasResult = false;
        	for(int i = 0; i < 16; i++)
        	{
        		int s1Add97 = getMS1IndexNotEqual0(a1s[i],i2);
        		if(s2.charAt(i2) == (char)s1Add97)
        		{
        			hasResult = true;
        			System.out.print(a1s[i]);
        			break;
        		}
        		
        	}
        	if(hasResult == false)
        		System.out.println("NO");
    	}
    }
    
    static int[] s1 = new int[100];
    
    //find dest[not equal 0]
    //mD = dest[i]
    public static int getMS1IndexNotEqual0(int mD, int i)
    {
    	if(i == 0)
    	{
    		return -1;
    	}
    	int v4 = jumble(mD);
    	int v5 = s1[i - 1] + v4;
    	int v6 = (v5 >> 31) >>> 28;
    	s1[i] = ((v6 + v5) & 0xF) - v6;
    	//s1[i] += 97;
    	return (s1[i]+97);
    }

Output:

    8c91cd2ff85e90efbe041faf4b572413470a5eb5f47ff9f13dec686b091e46829c55eff9d23ccdb53c0ea76b277b74ea836

This is the last 99 char of key

#### Conculusion of dest

The first char of key is "6",
So, the complete char of key is "68c91cd2ff85e90efbe041faf4b572413470a5eb5f47ff9f13dec686b091e46829c55eff9d23ccdb53c0ea76b277b74ea836"

#### Test the key

![congrats.png][2]


## Find Final Flag

The key of otp is "68c91cd2ff85e90efbe041faf4b572413470a5eb5f47ff9f13dec686b091e46829c55eff9d23ccdb53c0ea76b277b74ea836"
The contents in flag.txt is "18a07fbdbcd1af759895328ec4d82d2b411dc7876c34a0ab61eda8f2efa5bb0f198a3aa0ac47ff9a0cf3d913d3138678ce4b"
After xor them, the result is "7069636f4354467b63757374306d5f6a756d626c33735f3472336e745f345f67304f645f316433415f33336561643136667d"

Convert the xor result ("7069636f4354467b63757374306d5f6a756d626c33735f3472336e745f345f67304f645f316433415f33336561643136667d") from hex to ascii, it will be "picoCTF{cust0m_jumbl3s_4r3nt_4_g0Od_1d3A_33ead16f}", which is the final flag.

#### java code

    public static void main (String[] args) {
    	String str1 = "68c91cd2ff85e90efbe041faf4b572413470a5eb5f47ff9f13dec686b091e46829c55eff9d23ccdb53c0ea76b277b74ea836";
    	String str2 = "18a07fbdbcd1af759895328ec4d82d2b411dc7876c34a0ab61eda8f2efa5bb0f198a3aa0ac47ff9a0cf3d913d3138678ce4b";
    	String xorRes = doXor(str1,str2);
    	String flag = hexToString(xorRes);
    }
    
    public static String doXor(String str1, String str2)
    {
    	String xorRes = StringXor(str1, str2);
    	System.out.println(xorRes);
    	return xorRes;
    }
    
    public static String StringXor(String str1, String str2) {
    	BigInteger big1 = new BigInteger(str1, 16);
    	BigInteger big2 = new BigInteger(str2, 16);
    	return big1.xor(big2).toString(16);
    	
    }
    
    public static String hexToString(String hex)
    {
        byte[] s = DatatypeConverter.parseHexBinary(hex);
        String ret = new String(s);
        System.out.println(ret);
        return ret;
    }

Output:

    7069636f4354467b63757374306d5f6a756d626c33735f3472336e745f345f67304f645f316433415f33336561643136667d
    picoCTF{cust0m_jumbl3s_4r3nt_4_g0Od_1d3A_33ead16f}


  [1]: https://frc6.com/usr/uploads/2020/10/783240209.zip
  [2]: https://frc6.com/usr/uploads/2020/10/813829368.png


