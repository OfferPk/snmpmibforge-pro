/\*\*  
 \* SNMPMibForge Pro \- Core SNMPv3 Engine & Key Localization  
 \* Implements RFC 3411 (Architecture), RFC 3414 (USM & Key Localization),  
 \* RFC 7860 (HMAC-SHA-2 Protocols), and RFC 3415 (VACM).  
 \*/

class SNMPEngine {  
  constructor() {}

  /\*\*  
   \* Pure JS SHA-256 implementation for synchronous browser & test usage  
   \*/  
  sha256Bytes(bytes) {  
    if (typeof require \!== 'undefined') {  
      try {  
        const crypto \= require('crypto');  
        return crypto.createHash('sha256').update(Buffer.from(bytes)).digest();  
      } catch (e) {}  
    }

    // Pure JS SHA-256  
    function rightRotate(value, amount) {  
      return (value \>\>\> amount) | (value \<\< (32 \- amount));  
    }

    const mathPow \= Math.pow;  
    const maxWord \= mathPow(2, 32);  
    let i, j;  
    const result \= \[\];  
    const words \= \[\];  
    const asciiBitLength \= bytes.length \* 8;

    let hash \= \[\];  
    let k \= \[\];  
    let primeCounter \= 0;

    const isPrime \= {};  
    for (let candidate \= 2; primeCounter \< 64; candidate++) {  
      if (\!isPrime\[candidate\]) {  
        for (i \= 0; i \< 300; i \+= candidate) isPrime\[i\] \= true;  
        hash\[primeCounter\] \= (mathPow(candidate, 0.5) \* maxWord) | 0;  
        k\[primeCounter++\] \= (mathPow(candidate, 1 / 3\) \* maxWord) | 0;  
      }  
    }  
    hash \= hash.slice(0, 8);

    for (i \= 0; i \< bytes.length; i++) {  
      words\[i \>\> 2\] |= (bytes\[i\] & 0xff) \<\< (24 \- (i % 4\) \* 8);  
    }  
    words\[asciiBitLength \>\> 5\] |= 0x80 \<\< (24 \- (asciiBitLength % 32));  
    words\[(((asciiBitLength \+ 64\) \>\> 9\) \<\< 4\) \+ 15\] \= asciiBitLength;

    for (i \= 0; i \< words.length; i \+= 16\) {  
      const w \= words.slice(i, i \+ 16);  
      const oldHash \= hash.slice(0);

      for (j \= 0; j \< 64; j++) {  
        const w15 \= w\[j \- 15\], w2 \= w\[j \- 2\];  
        const s0 \= rightRotate(w15, 7\) ^ rightRotate(w15, 18\) ^ (w15 \>\>\> 3);  
        const s1 \= rightRotate(w2, 17\) ^ rightRotate(w2, 19\) ^ (w2 \>\>\> 10);  
        const ch \= (hash\[4\] & hash\[5\]) ^ (\~hash\[4\] & hash\[6\]);  
        const maj \= (hash\[0\] & hash\[1\]) ^ (hash\[0\] & hash\[2\]) ^ (hash\[1\] & hash\[2\]);  
        const temp1 \= (hash\[7\] \+ (rightRotate(hash\[4\], 6\) ^ rightRotate(hash\[4\], 11\) ^ rightRotate(hash\[4\], 25)) \+ ch \+ k\[j\] \+ (w\[j\] \= (j \< 16\) ? w\[j\] : (w\[j \- 16\] \+ s0 \+ w\[j \- 7\] \+ s1) | 0)) | 0;  
        const temp2 \= ((rightRotate(hash\[0\], 2\) ^ rightRotate(hash\[0\], 13\) ^ rightRotate(hash\[0\], 22)) \+ maj) | 0;

        hash \= \[(temp1 \+ temp2) | 0, hash\[0\], hash\[1\], hash\[2\], (hash\[3\] \+ temp1) | 0, hash\[4\], hash\[5\], hash\[6\]\];  
      }

      for (j \= 0; j \< 8; j++) hash\[j\] \= (hash\[j\] \+ oldHash\[j\]) | 0;  
    }

    const out \= \[\];  
    for (i \= 0; i \< 8; i++) {  
      for (j \= 3; j \>= 0; j--) {  
        out.push((hash\[i\] \>\> (8 \* j)) & 255);  
      }  
    }  
    return new Uint8Array(out);  
  }

  /\*\*  
   \* RFC 3414 / RFC 7860 Password-to-Key (P2K) Localization Algorithm  
   \* @param {string} password \- User passphrase  
   \* @param {string} engineIdHex \- Hexadecimal authoritative EngineID  
   \* @returns {string} Localized key in hex string  
   \*/  
  localizeKeySHA256(password, engineIdHex) {  
    if (\!password || \!engineIdHex) return '';

    // Convert password string to byte array  
    const pwBytes \= \[\];  
    for (let i \= 0; i \< password.length; i++) {  
      pwBytes.push(password.charCodeAt(i));  
    }

    // 1\. Expand password to 1,048,576 bytes (1MB)  
    const TARGET\_LEN \= 1048576;  
    const expanded \= new Uint8Array(TARGET\_LEN);  
    for (let i \= 0; i \< TARGET\_LEN; i++) {  
      expanded\[i\] \= pwBytes\[i % pwBytes.length\];  
    }

    // 2\. Compute master digest K1 \= SHA-256(expanded)  
    const k1 \= this.sha256Bytes(expanded);

    // 3\. Convert EngineID hex string to byte array  
    const cleanHex \= engineIdHex.replace(/\[^0-9a-fA-F\]/g, '');  
    const engineBytes \= \[\];  
    for (let i \= 0; i \< cleanHex.length; i \+= 2\) {  
      engineBytes.push(parseInt(cleanHex.substr(i, 2), 16));  
    }

    // 4\. Concatenate K1 \+ EngineID \+ K1  
    const concat \= new Uint8Array(k1.length \+ engineBytes.length \+ k1.length);  
    concat.set(k1, 0);  
    concat.set(engineBytes, k1.length);  
    concat.set(k1, k1.length \+ engineBytes.length);

    // 5\. Compute localized key \= SHA-256(K1 \+ EngineID \+ K1)  
    const localized \= this.sha256Bytes(concat);

    // Return as hex  
    let hex \= '';  
    for (let i \= 0; i \< localized.length; i++) {  
      hex \+= localized\[i\].toString(16).padStart(2, '0');  
    }  
    return hex;  
  }

  /\*\*  
   \* Formats EngineID into human-readable breakdown  
   \*/  
  parseEngineId(hexStr) {  
    const clean \= (hexStr || '').replace(/\[^0-9a-fA-F\]/g, '').toLowerCase();  
    if (clean.length \< 10\) {  
      return { valid: false, reason: 'EngineID too short (must be at least 10 hex characters / 5 octets)' };  
    }

    const firstByte \= parseInt(clean.substr(0, 2), 16);  
    const isEnterprise \= (firstByte & 0x80) \!== 0;

    let enterpriseNum \= 0;  
    let format \= 'unknown';

    if (isEnterprise) {  
      enterpriseNum \= parseInt(clean.substr(0, 8), 16\) & 0x7fffffff;  
      const formatByte \= parseInt(clean.substr(8, 2), 16);  
      switch (formatByte) {  
        case 1: format \= 'IPv4'; break;  
        case 2: format \= 'IPv6'; break;  
        case 3: format \= 'MAC Address'; break;  
        case 4: format \= 'Text string'; break;  
        case 5: format \= 'Octets'; break;  
        default: format \= 'Enterprise Specific'; break;  
      }  
    }

    return {  
      valid: true,  
      rawHex: clean,  
      isEnterprise,  
      enterpriseNum,  
      format,  
      octetLength: clean.length / 2  
    };  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= SNMPEngine;  
}  
