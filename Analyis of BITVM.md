## 1 Overview of BITVM

### 1.1 Code architecture design

### 1.2 Instructions for using the code

## 2 In-depth Analysis of BITVM Principles and Security 

### 2.1 Design and analysis of modules

#### 2.1.1 Big Integer

##### 2.1.1.1 mod crate

This code defines a struct BigIntImpl used to represent large integers, with generic parameters ```N_BITS``` and ```LIMB_SIZE``` to control the total number of bits and the size of each limb, respectively.

By adjusting the generic parameters, you can create integer types of different sizes and work with them conveniently.

**(1) Code analysis**

* Struct Definition:
  
```rust
pub struct BigIntImpl<const N_BITS: u32, const LIMB_SIZE: u32> {}
```

&emsp;&emsp;```BigIntImpl``` is a generic struct taking two constant generic parameters:

&emsp;&emsp;```N_BITS```: The total number of bits for the large integer.

&emsp;&emsp;```LIMB_SIZE```: The number of bits in each limb.

The struct itself has no data members, it's just a placeholder to define associated functions and constants.

* Associated Functions and Constants:

```rust
impl<const N_BITS: u32, const LIMB_SIZE: u32> BigIntImpl<N_BITS, LIMB_SIZE> {
    pub const N_BITS: u32 = N_BITS;
    pub const N_LIMBS: u32 = (N_BITS + LIMB_SIZE - 1) / LIMB_SIZE;
    pub const HEAD: u32 = N_BITS - (Self::N_LIMBS - 1) * LIMB_SIZE;
    pub const HEAD_OFFSET: u32 = 1u32 << Self::HEAD;
}
```

Associated functions and constants are defined within the impl block and are associated with the generic struct ```BigIntImpl```.

&emsp;&emsp;```N_BITS``` and ```LIMB_SIZE``` values are passed via generic parameters and used within associated functions and constants.

&emsp;&emsp;```N_LIMBS```: Calculates the number of limbs required to store the large integer.

&emsp;&emsp;```HEAD```: Calculates the number of significant bits in the first limb.

&emsp;&emsp;```HEAD_OFFSET```: Calculates the mask used to access the significant bits in the first limb.

#### 2.1.1.2 std crate

This code defines functions for performing standard operations on large integers represented by the BigIntImpl struct.

**(1) Code analysis**

```rust
push_u32_le(v: &[u32])
```

This function converts an array of u32 values ```v``` into a code block for the scripting language and pushes it onto the stack.

It first converts each u32 value in ```v``` into its binary representation and stores them in a ```Vec<bool>```.

Then, it divides the bits in the ```Vec<bool>``` into multiple limbs, where each limb has ```LIMB_SIZE``` bits.

Finally, it converts each limb value back into a u32 type, stores them in a ```Vec<u32>```, and pushes these limb values onto the stack one by one.

```rust
push_u64_le(v: &[u64])
```

This function converts an array of u64 values ```v``` into a code block for the scripting language and pushes it onto the stack.

It first splits each u64 value into two u32 values and stores them in a ```Vec<u32>```.

Then, it calls the ```push_u32_le``` function to convert the ```Vec<u32>``` into a script code block and push it onto the stack.

```rust
zip(mut a: u32, mut b: u32)
```

This function interleaves the limbs of two large integers ```a``` and ```b```, and pushes the result onto the stack.

It first calculates the starting indices for the two large integers.

Then, it interleaves the limbs of the two large integers based on the starting indices and pushes them onto the stack.

```rust
copy_zip(mut a: u32, mut b: u32)
```

This function copies and interleaves the limbs of two large integers ```a``` and ```b```, and pushes the result onto the stack.

It first calculates the starting indices for the two large integers.

Then, it copies the limbs of the two large integers based on the starting indices, interleaves them, and pushes them onto the stack.

```rust
dup_zip(mut a: u32) 
```
This function duplicates and interleaves the limbs of a large integer ```a```, and pushes the result onto the stack.

It first calculates the starting index for the large integer.

Then, it duplicates each limb of the large integer based on the starting index, interleaves them, and pushes them onto the stack.

```rust
copy(mut a: u32) 
```

This function copies all the limbs of a large integer ```a``` onto the stack.

It uses the ```OP_PICK``` operator to copy each limb to the top of the stack based on the starting index of the large integer.

```rust
roll(mut a: u32)
```

This function rotates all the limbs of a large integer ```a``` by a specified number of positions, and pushes the result onto the stack.

It uses the ```OP_ROLL``` operator to rotate each limb by the specified number of positions based on the starting index of the large integer.

```rust
drop()
```

This function removes the large integer ```a``` at the top of the stack.

It uses the ```OP_2DROP``` operator to remove two limbs at a time until all limbs are removed.

```rust
push_dec(dec_string: &str)
```

This function converts a decimal string ```dec_string``` into a code block for the scripting language and pushes it onto the stack.

It uses the BigUint library to convert the decimal string into a large integer, then uses the ```to_u32_digits``` function to convert the large integer into a u32 array, and finally calls the ```push_u32_le``` function to push the u32 array onto the stack.

```rust
push_hex(hex_string: &str)
```

This function converts a hexadecimal string ```hex_string``` into a code block for the scripting language and pushes it onto the stack.

It uses the BigUint library to convert the hexadecimal string into a large integer, then uses the ```to_u32_digits``` function to convert the large integer into a u32 array, and finally calls the ```push_u32_le``` function to push the u32 array onto the stack.

```rust
push_zero()
```

This function pushes a large integer with a value of 0 onto the stack.

It calls the ```push_to_stack``` function to push the value 0 onto the stack ```N_LIMBS``` times.

```rust
push_one() 
```
This function pushes a large integer with a value of 1 onto the stack.

It calls the ```push_to_stack``` function to push the value 0 onto the stack ```N_LIMBS-1``` times, and then pushes the value 1 onto the stack.

```rust
is_zero_keep_element(a: u32)
```

This function checks if a large integer ```a``` is equal to 0 and pushes the result onto the stack.

It uses a loop to check if each limb of the large integer ```a``` is equal to 0, and accumulates the result using a logical ```AND``` operation.

```rust
is_zero(a: u32) 
```

This function checks if a large integer ```a``` is equal to 0 and pushes the result onto the stack.

It uses a loop to check if each limb of the large integer ```a``` is equal to 0, and accumulates the result using a logical ```AND``` operation.

```rust
is_one_keep_element(a: u32) 
```

This function checks if a large integer ```a``` is equal to 1 and pushes the result onto the stack.

It uses a loop to check if each limb of the large integer ```a``` is equal to 1, and accumulates the result using a logical ```AND``` operation.

```rust
is_one(a: u32) 
```

This function checks if a large integer ```a``` is equal to 1 and pushes the result onto the stack.

It uses a loop to check if each limb of the large integer ```a``` is equal to 1, and accumulates the result using a logical ```AND``` operation.

```rust
toaltstack() 
```

This function moves all the limbs of a large integer ```a``` from the main stack to the auxiliary stack.

It uses the ```OP_TOALTSTACK``` operator to move each limb to the auxiliary stack.

```rust
fromaltstack()
```

This function moves all the limbs of a large integer a from the auxiliary stack to the main stack.

It uses the ```OP_FROMALTSTACK``` operator to move each limb to the main stack.

**(2) Optimization**

This code implements a way to handle large integers by dividing them into multiple limbs and manipulating these limbs through a series of operations, ultimately pushing the result onto the stack for use in scripting languages. This approach effectively handles very large integers without being limited by the size of integer types.

#### 2.1.1.3 add crate

This code defines functions for performing addition operations on large integers represented by the BigIntImpl struct. 

**(1) Code analysis**

```rust
double(a: u32)
```

This function doubles a large integer represented by ```a```. Here's the breakdown:

* ``` Self::dup_zip(a)```: This call duplicates and "zips" a u32 value representing ```a``` large integer, preparing it for subsequent large integer doubling.

* ```1 << LIMB_SIZE```: This pushes the value 2^```LIMB_SIZE```, which is the base for each limb.

* ```limb_add_carry OP_TOALTSTACK```: Adds the first limbs (A0 and B0) and handles a possible carry bit. The result is pushed to the alternate stack.

* ```for _ in 0..Self::N_LIMBS - 2 { ... }```: This loop iterates over the remaining limbs (A1 to A{N-2}) and B limbs. In each iteration:

&emsp;&emsp;```OP_ROT```: Rotates the top three stack items.

&emsp;&emsp;```OP_ADD```: Adds the current limbs (A{i} and B{i}) and the carry bit from the previous addition.

&emsp;&emsp;```OP_SWAP```: Swaps the top two stack items.

&emsp;&emsp;```limb_add_carry OP_TOALTSTACK```: Performs limb addition with carry and pushes the result to the alternate stack.


* ```OP_NIP OP_ADD { limb_add_nocarry(Self::HEAD_OFFSET) }```: Adds the last two limbs (A{N-1} and B{N-1}) with the carry bit, and handles the carry bit for the head limb.

* ```for _ in 0..Self::N_LIMBS - 1 { OP_FROMALTSTACK }```: This loop retrieves all the calculated limbs from the alternate stack and places them on the main stack, resulting in the doubled large integer.

```rust
add(a: u32, b: u32)
```

This function adds two large integers represented by ```a``` and ```b```. It follows a similar structure to double, with the key difference being the addition of limbs from two separate integers instead of doubling a single integer.

```rust
add1()
```

This function adds 1 to the current large integer. It's optimized for adding 1, only using the ```limb_add_carry``` operation once for the first limb and then handling carry bits for the remaining limbs.

```rust
limb_add_carry()
```

This function implements limb addition with carry. It takes two limbs and the base value (2^```LIMB_SIZE```) as input and returns the sum of the limbs, taking care of the carry bit.

```rust
limb_add_nocarry(head_offset: u32)
```

This function implements limb addition without carry. It takes two limbs and a head_offset (which is the mask for the head limb) as input. It adds the limbs and handles the carry bit for the head limb.

**(2) Optimization**

* ```limb_add_carry``` and ```limb_add_nocarry``` are optimized for efficiency by leveraging the stack operations of the underlying script language.

* The for loops in double, add, and add1 are used to efficiently handle multiple limbs.

#### 2.1.1.4 sub crate

This code defines functions for performing subtraction operations on large integers represented by the BigIntImpl struct. 

**(1) Code analysis**
```rust
sub(a: u32, b: u32)
```

This function uses the script! macro to calculate the difference between two large integers ```a``` and ```b```. Here's the breakdown:

* ```Self::zip(a, b)```: Interleaves the limbs of the two large integers ```a``` and ```b``` and pushes the result onto the stack.

* ```1 << LIMB_SIZE```: Pushes 2^```LIMB_SIZE``` onto the stack, which is the base for each limb.

* ```limb_sub_borrow OP_TOALTSTACK```: Calculates the difference of the first limb (A0 - B0) and handles borrowing. The result is pushed onto the auxiliary stack.

* Loop ```for _ in 0..Self::N_LIMBS - 2 { ... }```: Iterates through the remaining limbs, calculating the difference of each limb and handling borrowing.

* ```OP_NIP OP_ADD { limb_sub_noborrow(Self::HEAD_OFFSET) }```: Calculates the difference of the last limb and handles borrowing.

* Loop ```for _ in 0..Self::N_LIMBS - 1 { OP_FROMALTSTACK }```: Takes all the calculated limbs from the auxiliary stack and places them on the main stack.

```rust
limb_sub_borrow()
```
This function calculates the difference between two limbs and handles borrowing.

It uses a series of stack operations (e.g., ```OP_ROT```, ```OP_SUB```, ```OP_DUP```, ```OP_LESSTHAN```, ```OP_TUCK```, ```OP_IF```, ```OP_PICK```, ```OP_ADD```, ```OP_ENDIF```) to perform the calculation and handle borrowing.

```rust
 limb_sub_noborrow(head_offset: u32)
```

This function calculates the difference between two limbs but does not handle borrowing.

It uses a series of stack operations to perform the calculation and decides whether to add ```head_offset``` based on whether the result is less than 0.

**(2) Optimization**

The code uses an algorithm similar to manual subtraction to calculate the difference between two large integers. It first interleaves the limbs of the two large integers, then calculates the difference limb by limb, handling borrowing. Each limb difference calculation is done using the limb_sub_borrow or limb_sub_noborrow function, which handle borrowing and no borrowing, respectively. Finally, it places all the calculated limbs on the main stack, resulting in the difference between the two large integers.

The operators ```OP_ROLL```, ```OP_PICK```, ```OP_DUP```, ```OP_SUB```, ```OP_ADD```, ```OP_LESSTHAN```, ```OP_TUCK```, ```OP_IF```, ```OP_ENDIF```, etc., in the code are stack operators in scripting languages used to manipulate elements on the stack.

#### 2.1.1.5 mul crate

This code defines functions for performing multiplication operations on large integers represented by the BigIntImpl struct. 

**(1) Code analysis**

```rust
mul() 
```

This function uses the script! macro to perform multiplication operations on large integers. Here's the breakdown:

* ```{ Self::convert_to_be_bits_toaltstack() }```: Converts the two large integers on the top of the stack to binary form and pushes them onto the auxiliary stack.

* ```{ push_to_stack(0,Self::N_LIMBS as usize) }```: Pushes a large integer with a value of 0 to the top of the stack.

* ```OP_FROMALTSTACK```: Takes the large integer from the top of the auxiliary stack and places it on the top of the main stack.

* ```OP_IF ... OP_ENDIF```: A conditional statement that executes ```Self::copy(1)``` and ```Self::add(1, 0)``` if the large integer at the top of the auxiliary stack is not 0; otherwise, it does nothing.

* ```Loop for _ in 1..N_BITS - 1 { ... }```: Iterates through each bit of the large integer and performs the following operations:

&emsp;&emsp;```{ Self::roll(1) }```: Moves the element at the top of the stack down by one position.

&emsp;&emsp;```{ Self::double(0) }```: Multiplies the element at the top of the stack by 2.

&emsp;&emsp;```{ Self::roll(1) }```: Moves the element at the top of the stack down by one position.

&emsp;&emsp;```OP_FROMALTSTACK```: Takes the large integer from the top of the auxiliary stack and places it on the top of the main stack.

&emsp;&emsp;```OP_IF ... OP_ENDIF```: A conditional statement that executes ```Self::copy(1)``` and ```Self::add(1, 0)``` if the large integer at the top of the auxiliary stack is not 0; otherwise, it does nothing.

* ```{ Self::roll(1) }```: Moves the element at the top of the stack down by one position.

* ```{ Self::double(0) }```: Multiplies the element at the top of the stack by 2.

* ```OP_FROMALTSTACK```: Takes the large integer from the top of the auxiliary stack and places it on the top of the main stack.

* ```OP_IF ... OP_ELSE ... OP_ENDIF```: A conditional statement that executes ```Self::add(1, 0)``` if the large integer at the top of the auxiliary stack is not 0; otherwise, it executes ```Self::drop()```.

**(2) Optimization**

The code uses an algorithm similar to binary multiplication to calculate the product of two large integers. It first converts the two large integers to binary form and uses a large integer with a value of 0 as the result. Then, it iterates through each bit of the second large integer, and if the current bit is 1, it multiplies the first large integer by 2 and adds it to the result. Finally, it returns the result.

The operators ```OP_ROLL```, ```OP_FROMALTSTACK```, ```OP_IF```, ```OP_ELSE```, ```OP_ENDIF```, ```OP_TOALTSTACK```, ```OP_DROP```, ```OP_ADD```, etc., are stack operators in scripting languages that manipulate elements on the stack.

#### 2.1.1.6 inv crate

This code defines several functions for performing division and modular inversion operations on large integers represented by the BigIntImpl struct.  

**(1) Code analysis**

```rust
div2()
div2rem()
```

The ```div2``` function divides a large integer by 2 and pushes the result onto the stack. Here's the breakdown:

* ```{ Self::div2rem() }```: Calls the ```div2rem``` function, which calculates the quotient and remainder of the division by 2.
* ```OP_DROP```: Discards the remainder at the top of the stack, leaving only the quotient.

The ```div2rem``` function divides a large integer by 2 and pushes both the quotient and remainder onto the stack. Here's the breakdown:

* ```{ Self::N_LIMBS - 1 } OP_ROLL```: Moves the last limb at the top of the stack to the bottom, and other limbs move up.

* ```0```: Pushes the value 0 to the top of the stack.

* ```{ limb_shr1_carry(Self::HEAD) }```: Performs a right-shift by one bit operation on the head limb, handling carry.

* Loop ```for _ in 1..Self::N_LIMBS { ... }```: Iterates through all limbs except the head limb.

* ```{ Self::N_LIMBS } OP_ROLL```: Moves the last limb at the top of the stack to the bottom, and other limbs move up.

* ```OP_SWAP```: Swaps the top two elements on the stack.

* ```{ limb_shr1_carry(LIMB_SIZE) }```: Performs a right-shift by one bit operation on the current limb, handling carry.

```rust
div3()
div3rem()
```

The ```div3``` function divides a large integer by 3 and pushes the result onto the stack.Here's the breakdown:

* ```{ Self::div3rem() }```: Calls the ```div3rem``` function, which calculates the quotient and remainder of the division by 3.
* ```OP_DROP```: Discards the remainder at the top of the stack, leaving only the quotient.

The ```div3rem``` function divides a large integer by 3 and pushes both the quotient and remainder onto the stack. Here's the breakdown:

* ```{ Self::N_LIMBS - 1 } OP_ROLL```: Moves the last limb at the top of the stack to the bottom, and other limbs move up.

* ```0```: Pushes the value 0 to the top of the stack.

* ```{ limb_div3_carry(Self::HEAD) }```: Performs a division by 3 operation on the head limb, handling carry..

* Loop ```for _ in 1..Self::N_LIMBS { ... }```: Iterates through all limbs except the head limb.

* ```{ Self::N_LIMBS } OP_ROLL```: Moves the last limb at the top of the stack to the bottom, and other limbs move up.

* ```OP_SWAP```: Swaps the top two elements on the stack.

* ```{ limb_div3_carry(LIMB_SIZE) }```: Performs a division by 3 operation on the current limb, handling carry.

```rust
inv_stage1()
inv_stage2(modulus_hex: &str)
```
The ```inv_stage1``` function is the first stage of the Constant Time Modular Inversion algorithm(An algorithm for calculating modular inverses whose computation time is independent of the input, avoiding timing attacks.). It uses a variant of the extended Euclidean algorithm to calculate the modular inverse. This stage performs a series of operations, ultimately resulting in an intermediate value related to the modular inverse.

The ```inv_stage2``` function is the second stage of the Constant Time Modular Inversion algorithm. It uses the BigUint library to calculate the final result of the modular inverse.

Assuming the large integers stored in BigIntImpl are ```u```, ```r```, ```v```, ```s```, and ```k```, after calling the ```inv_stage1``` function, the main stack will contain ```s``` and ```k```, and the auxiliary stack will be empty. You can then use the ```inv_stage2``` function to calculate the final modular inverse.

```rust
limb_shr1_carry(num_bits: u32) 
```

This function right-shifts a limb by one bit and handles carrying.Here's the breakdown:

* ```powers_of_2_script``` section: This code defines a script code block named ```powers_of_2_script``` which generates a series of powers of 2 values. If num_bits is less than 7, it generates values from 2^1 to 2^```{num_bits - 1}``` using a loop. Otherwise, it directly generates values from 2^1 to 2^6, and then uses a loop to generate values from 2^7 to 2^```{num_bits - 1}```.

* Main Logic Section: This code first pushes ```powers_of_2_script``` to the top of the stack and moves ```num_bits - 1``` to the top of the stack. Then, it uses ```OP_IF``` and ```OP_ELSE``` operators to handle the case where ```num_bits - 1``` is zero or not. After that, it moves ```num_bits - 1``` to the top of the stack again and performs a loop. Each iteration uses the ```OP_2DUP``` operator to duplicate the top two elements on the stack and uses different operations based on the comparison result. After the loop ends, it uses ```OP_IF``` and ```OP_ELSE``` operators again to handle the result of the last comparison and pushes the final quotient and remainder onto the stack.

```rust
limb_div3_carry(limb_size: u32)
```

This function divides a limb by 3 and pushes both the quotient and remainder onto the stack. Here's the breakdown:

* Preprocessing Section ```let max_limb = (1 << limb_size) as i64; ...; while cur < max_limb {...};```: Performs the following operations:

&emsp;&emsp;```max_limb```:  Calculates the maximum value of a limb, which is 2^```limb_size```.

&emsp;&emsp; ```x_quotient, x_remainder, y_quotient, y_remainder```: Pre-calculate some quotients and remainders of division by 3 for later use in the script code.

&emsp;&emsp; ```k```: Used to calculate powers of 3 for later use in the script code.

&emsp;&emsp; ```cur```: Used to calculate powers of 3 until it becomes greater than ```max_limb```.

* Script Code Section: This code uses a series of stack operations to perform the division by 3 operation. First, it pushes the powers of 3 (1, 2, 3, 6, 9, 18, 27, 54) onto the stack and uses a loop to extend them to ```2 * k``` elements. Then, it duplicates the top element on the stack (```2 * k``` powers of 3) and uses comparison operations to determine which power of 3 is closest to the current limb value. Based on the comparison result, it pushes the corresponding quotient and remainder onto the stack and pushes them to the auxiliary stack. Next, it moves the top element on the stack (```2 * k + 1``` powers of 3) to the top of the stack and uses a loop to perform subtraction operations. Finally, it retrieves the quotient and remainder from the auxiliary stack and places them on the main stack, performing some stack operations to adjust the stack state.

**(2) Optimization**

This code implements a series of functions related to large integer division and modular inversion calculation. They perform division and modular inversion by processing operations on each limb and use stack operations in scripting languages to implement these operations.

The operators ```OP_ROLL```, ```OP_SWAP```, ```OP_DROP```, ```OP_TOALTSTACK```, ```OP_FROMALTSTACK```, ```OP_IF```, ```OP_ELSE```, ```OP_ENDIF```, ```OP_ADD```, ```OP_SUB```, ```OP_DUP```, ```OP_NOT```, ```OP_NEGATE```, ```OP_PICK```, ```OP_GREATERTHAN```, ```OP_LESSTHANOREQUAL```, etc., in the code are stack operators in scripting languages used to manipulate elements on the stack.

#### 2.1.1.7 cmp crate

This code defines several functions for comparing two large integers represented by the BigIntImpl struct.  

**(1) Code analysis**

```rust
equalverify(a: u32, b: u32) 
```
This function checks if two large integers ```a``` and ```b``` are equal.

It uses the``` Self::zip``` function to interleave the limbs of the two large integers and then compares each limb using the ```OP_EQUALVERIFY``` operator.

```rust
equal(a: u32, b: u32) 
```

This function checks if two large integers ```a``` and ```b``` are equal and pushes the result onto the stack.

It uses the ```Self::zip``` function to interleave the limbs of the two large integers and then compares each limb using the ```OP_EQUAL``` operator, pushing the result onto the auxiliary stack. Finally, it retrieves all the results from the auxiliary stack and uses the ```OP_BOOLAND``` operator to perform a logical ```AND``` operation to get the final comparison result.

```rust
notequal(a: u32, b: u32) 
```

This function checks if two large integers ```a``` and ```b``` are not equal and pushes the result onto the stack.

It calls the ```Self::equal``` function to check if the two large integers are equal and negates the result.

```rust
lessthan(a: u32, b: u32) 
```

This function checks if the large integer ```a``` is less than the large integer ```b``` and pushes the result onto the stack.

It uses the ```Self::zip``` function to interleave the limbs of the two large integers and then compares using the ```OP_GREATERTHAN``` and ```OP_LESSTHAN``` operators, pushing the result onto the auxiliary stack. Then, it uses the ```OP_BOOLOR``` and ```OP_BOOLAND``` operators to combine the comparison results of each limb to get the final comparison result.

```rust
lessthanorequal(a: u32, b: u32) 
```

This function checks if the large integer ```a``` is less than or equal to the large integer ```b``` and pushes the result onto the stack.

It calls the ```Self::greaterthanorequal``` function and swaps the parameters.

```rust
greaterthan(a: u32, b: u32) 
```

This function checks if the large integer ```a``` is greater than the large integer ```b``` and pushes the result onto the stack.

It calls the ```Self::lessthanorequal``` function and negates the result.

```rust
greaterthanorequal(a: u32, b: u32) 
```

This function checks if the large integer ```a``` is greater than or equal to the large integer ```b``` and pushes the result onto the stack.

It calls the ```Self::lessthan``` function and negates the result.

**(2) Optimization**

This code implements a series of functions to compare two large integers. These functions use the Self::zip function to interleave the limbs of the two large integers and then compare each limb. They use different stack operators to perform comparison operations and use the auxiliary stack to store intermediate results as needed.

#### 2.1.1.8 bits crate

This code defines several functions that convert the individual limbs of a large integer into their binary bit representation, adjusting the bit order (big-endian or little-endian) as needed.

**(1) Code analysis**

*Conversion Functions*

```rust
convert_to_be_bits();
convert_to_le_bits()
```
These two functions convert the large integer to big-endian and little-endian binary bit representation, respectively. They both use the helper functions ```limb_to_be_bits``` and ```limb_to_le_bits``` to handle the conversion of each limb.

```rust
convert_to_be_bits_toaltstack();
convert_to_le_bits_toaltstack()
```

These two functions are similar to the previous two functions, but they push the converted binary bits onto the auxiliary stack. They use the helper functions ```limb_to_be_bits_toaltstack``` and ```limb_to_le_bits_toaltstack```, respectively, to handle the conversion of each limb.

*Helper Functions*

```rust
limb_to_be_bits_common(num_bits: u32)
```

This function converts a limb to big-endian binary bits and pushes it onto the stack. It uses a loop to push powers of 2 onto the stack and then uses comparison operations to determine the value of each bit and push the result onto the auxiliary stack. Here's the breakdown:

* ```min_i```: Calculates the minimum value between ```num_bits - 1``` and 22. This is used to determine how many bytes are needed to represent the powers of 2.

*  ```script! {...}```: This part of the code pushes powers of 2 onto the stack and uses the auxiliary stack for operations. First, it pushes the values 2^0 to 2^```{min_i - 1}``` onto the auxiliary stack (because these values can be represented using 3 bytes). Then, it uses the ```OP_DUP``` operator to duplicate the top element on the stack and adds the duplicated element to generate values from 2^```{min_i}``` to 2^```{num_bits - 2}```, pushing these values onto the auxiliary stack. Finally, it pops all elements from the auxiliary stack and pushes them onto the main stack.

* ```for _ in 0..num_bits - 2 {...}; ...; OP_ENDIF```: This part of the code uses a loop to compare each bit of the limb with the powers of 2 and pushes the comparison result onto the auxiliary stack. It uses the ```OP_2DUP``` operator to duplicate the top two elements on the stack, uses the ```OP_LESSTHANOREQUAL``` operator for comparison, and performs different operations based on the comparison result. If the value of the current bit is greater than or equal to the power of 2, it pushes 1 to the auxiliary stack and subtracts the power of 2 from the current bit. Otherwise, it pushes 0 to the auxiliary stack.

```rust
limb_to_le_bits_common(num_bits: u32)
```

This function converts a limb to little-endian binary bits and pushes it onto the stack. It is similar to ```limb_to_be_bits_common``` but pushes the powers of 2 onto the auxiliary stack and pushes the comparison result onto the main stack. Here's the breakdown:

* ```min_i```: Calculates the minimum value between ```num_bits - 1``` and 22. This is used to determine how many bytes are needed to represent the powers of 2.

*  ```script! {...}```: This part of the code pushes powers of 2 onto the auxiliary stack and uses some stack operators to adjust the stack state. First, it pushes the values 2^0 to 2^```{min_i - 2}``` onto the auxiliary stack (because these values can be represented using 3 bytes). Then, if ```num_bits - 1``` is greater than ```min_i```, it duplicates 2^```{min_i - 1}``` and uses a loop to double it, generating values from 2^```{min_i}``` to 2^```{num_bits - 3}``` and pushing these values onto the auxiliary stack. Finally, it pushes 2^```{num_bits - 2}``` to the auxiliary stack as well. Otherwise, it only pushes 2^```{min_i - 1}``` to the auxiliary stack.

* ```for _ in 0..num_bits - 2 {...}; ...; OP_SWAP```: This part of the code uses a loop to compare each bit of the limb with the powers of 2 and pushes the comparison result onto the main stack. It uses the ```OP_FROMALTSTACK``` operator to pop the top element from the auxiliary stack and push it onto the main stack. Then, it uses the ```OP_2DUP``` operator to duplicate the top two elements on the stack and uses the ```OP_GREATERTHANOREQUAL``` operator for comparison, performing different operations based on the comparison result. If the value of the current bit is greater than or equal to the power of 2, it pushes 1 to the main stack and subtracts the power of 2 from the current bit. Otherwise, it pushes 0 to the main stack.

```rust
limb_to_le_bits(num_bits: u32)
```

This function converts a limb to little-endian binary bits and pushes it onto the stack. If ```num_bits``` is greater than or equal to 2, it calls the ```limb_to_le_bits_common``` function to perform the conversion.

```rust
limb_to_le_bits_toaltstack(num_bits: u32)
```

 This function converts a limb to little-endian binary bits and pushes it onto the auxiliary stack. If ```num_bits``` is greater than or equal to 2, it calls the ```limb_to_le_bits_common``` function to perform the conversion and pushes the result onto the auxiliary stack.

```rust
limb_to_be_bits(num_bits: u32)
```

This function converts a limb to big-endian binary bits and pushes it onto the stack. If ```num_bits``` is greater than or equal to 2, it calls the ```limb_to_be_bits_common``` function to perform the conversion.

```rust
limb_to_be_bits_toaltstack(num_bits: u32)
```

This function converts a limb to big-endian binary bits and pushes it onto the auxiliary stack. If ```num_bits``` is greater than or equal to 2, it calls the ```limb_to_be_bits_common``` function to perform the conversion and pushes the result onto the auxiliary stack.

**(2) Optimization**

These functions implement a way to convert large integers into their binary bit representation. They achieve this by processing the conversion of each limb and pushing the result onto the stack or the auxiliary stack. 

#### 2.1.1.9 u29x9 crate

The code defines a series of functions to compute the product of two 254-bit big integers, using the Karatsuba algorithm to optimize the computation process.

**(1) Code analysis**



**(2) Optimization**




**(1) Code analysis**

### 2.2 Module testing

### 2.3 Security analysis

## 3 Development process of BITVM


## 4 Development process of BITVM

### 4.1 Development examples of BITVM

## 5 Code maintenance