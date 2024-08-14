# Carry_Look-Ahead-_Adder_RISCV


A carry-look ahead adder (CLA) or fast adder is a type of adder used in digital logic. A carry-look ahead adder improves speed by reducing the amount of time required to determine to carry bits. It calculates one or more carries before the sum, which reduces the wait time/Ripple delay to calculate the result of the larger value bits of the adder.

**The 4-bit carry look-ahead carry adder is shown in the figure given below:**

![CLA_1](https://github.com/user-attachments/assets/bf4728c5-288f-4080-a636-bd8c107ac929)


### Carry Propagation Delay in Adders

Adders often face carry propagation delays during arithmetic operations like multiplication and division, as these operations involve multiple addition or subtraction steps. This issue is significant because enhancing the speed of addition can improve the performance of all other arithmetic operations. Therefore, minimizing the carry propagation delay in adders is crucial. Various logic design techniques have been developed to address this problem. One commonly used approach is the carry look-ahead method, which addresses the issue by calculating the carry signals in advance based on the input signals. This type of adder circuit is known as a **carry look-ahead adder**.

### Carry Signal Generation

A carry signal will be generated in two scenarios:
1. When both input bits `A` and `B` are 1.
2. When one of the two bits is 1 and the carry-in is also 1.

In ripple carry adders, although the two bits to be added in each adder block are available immediately, each adder block must wait for the carry to propagate from the previous block. Consequently, it is not possible to generate the sum and carry of any block until the input carry is known. The `i^{th}` block must wait for the `i-1^{th}` block to produce its carry, resulting in a significant time delay, known as **carry propagation delay**.

**The following shows the block diagram of 4-bit carry look-ahead adder:**

![BLOCK_DGM_1](https://github.com/user-attachments/assets/673adda9-a045-481e-a10b-e870135fec14)


### Ripple Carry Adder Example

Consider a 4-bit ripple carry adder. The sum `S_{3}` is generated by the corresponding full adder as soon as the input signals are applied. However, the carry input `C_{4}` does not reach its final steady-state value until carry `C_{3}` has reached its steady-state value. Similarly, `C_{3}` depends on `C_{2}`, and `C_{2}` on `C_{1}`. As a result, the carry must propagate through all the stages for the output `S_{3}` and carry `C_{4}` to settle at their final values.

The total propagation time is the product of the propagation delay of each adder block and the number of adder blocks in the circuit. For example, if each full adder stage has a propagation delay of 20 nanoseconds, then `S_{3}` will reach its correct value after 60 (20 × 3) nanoseconds. The situation becomes more challenging as the number of stages increases to handle more bits.

### Carry Look-ahead Adder

A carry look-ahead adder mitigates the propagation delay by introducing more complex circuitry. In this design, the ripple carry design is modified so that the carry logic over fixed groups of bits is reduced to two-level logic.

**The below shows two-level logic circuit diagram:**

![ckt_dgm_1](https://github.com/user-attachments/assets/5edca1a7-34ec-46dd-9dab-fed9845275b6)

### Full Adder Circuit Analysis

Consider the full adder circuit, which includes a corresponding truth table. We define two variables: ‘carry generate’ `G_{i}` and ‘carry propagate’ `P_{i}`. They are defined as follows:

![Truth_table_1](https://github.com/user-attachments/assets/ab8ca1a7-47af-4480-837b-37f28ee7620d)

\[
P_{i} = A_{i} \oplus B_{i} \newline G_{i} = A_{i} B_{i}
\]

The sum output and carry output can then be expressed in terms of carry generate `G_{i}` and carry propagate `P_{i}` as:

\[
S_{i} = P_{i} \oplus C_{i} \newline C_{i+1} = G_{i} + P_{i} C_{i}
\]

Here, `G_{i}` generates the carry when both `A_{i}` and `B_{i}` are 1, regardless of the input carry. `P_{i}` is responsible for propagating the carry from `C_{i}` to `C_{i + 1}`.

### Carry Output Boolean Function

The carry output Boolean function for each stage in a 4-stage carry look-ahead adder can be written as:

\[
C_{1} = G_{0} + P_{0} C_{in} \newline 
C_{2} = G_{1} + P_{1} C_{1} = G_{1} + P_{1} G_{0} + P_{1} P_{0} C_{in} \newline 
C_{3} = G_{2} + P_{2} C_{2} = G_{2} + P_{2} G_{1} + P_{2} P_{1} G_{0} + P_{2} P_{1} P_{0} C_{in} \newline 
C_{4} = G_{3} + P_{3} C_{3} = G_{3} + P_{3} G_{2} + P_{3} P_{2} G_{1} + P_{3} P_{2} P_{1} G_{0} + P_{3} P_{2} P_{1} P_{0} C_{in}
\]

##C-CODE

 ```bash
#include< stdio.h>
#include< conio.h>
#include< process.h>
#include< math.h>

int get1(int a)
{
char ch='B';
if(a==1)
ch='A';
do
{
printf("\n\tENTER VALUE OF %c:",ch);
scanf("%d",&a);
if(a< =0)
printf("\n\t\t!INVALID NUMBER.ENTER VALUE (0< A)!");
}while(a< =0);
return(a);
}

int and(int a,int b)
{
int c;
if(a< b)
c=a;
else
c=b;
return (c);
}

int or(int a,int b)
{
int x;
if(a>b)
x=a;
else
x=b;
return x;
}

int exor(int a,int b)
{
int x;
if(a==b)
x=0;
else
x=1;
return x;
}


void add()
{
int i=7,A,B,a,b,cin,num;
int n1[8],n2[8],cg[8],cp[8],sum[8];
for(i=0;i< =7;i++)
{
n1[i]=0; // Num 1
n2[i]=0; // Num 2
cg[i]=0; // Gi
cp[i]=0; // Pi
sum[i]=0; // Sum
}
A = a = get1(1);
B = b = get1(0);
i=7;
do
{
n1[i]=a%2;
a=a/2;
n2[i]=b%2;
b=b/2;
i--;
}while((a!=0)||(b!=0));
i=0;
printf("\n\t\t Binary Form",A);
printf("\n\t A = %d : ",A);
for(i=0;i< =7;i++)
printf("%d ",n1[i]);
printf("\n\t B = %d : ",B);
for(i=0;i< =7;i++)
printf("%d ",n2[i]);
cin=0;
for(i=7;i>=0;i--)
{
sum[i]=exor(cin,exor(n1[i],n2[i])); // Sum Pi (+) Bi
cg[i]=and(n1[i],n2[i]); // Gi = Ai . Bi
cp[i]=or(n1[i],n2[i]); // Pi = Ai (+) Bi
cin=or(cg[i],and(cp[i],cin)); // Cin =Gi + PiCi
}
printf("\n\n\t\t SUM: ");
num=0;
for(i=0;i< =7;i++)
{
printf(" %d",sum[i]);
num=num + (sum[i]*pow(2,7-i));
}
printf("\n\n\t\t SUM: %d + %d= %d\n",A,B,num);
printf("\t\t The Carry Is : %d\n\n",cin);
}

void main()
{
int ch,a,b,c,d;
clrscr();
while(1)
{
M: printf("******** MENU FOR LOOK AHEAD CARRY ADDER ********");
printf("\n\t\t1.ADDITION OF TWO NUMBER");
printf("\n\t\t2.EXIT\n");
printf("*************************************************");
printf("\n\t\tEnter Your Option:");
scanf("%d",&ch);
switch(ch)
{
case 1:
add();
getch();
break;

case 2: exit(0);
break;

default:
clrscr();
printf("ERROR!!!!!!!!! INVALID ENTRY...\n");
printf("Back To Main Menu\n\n");
goto M;
}
}
}

```
##VERIFICATION OF C CODE:

