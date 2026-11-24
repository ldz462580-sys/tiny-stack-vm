import * as fs from 'node:fs';

const POP: Record<number, number> = {};
const PSH: Record<number, number> = {};
const IMM: Record<number, number> = {};
POP[0] = 0; PSH[0] = 0; IMM[0] = 0;
POP[1] = 0; PSH[1] = 1; IMM[1] = 1;
POP[2] = 0; PSH[2] = 1; IMM[2] = 4;
POP[3] = 1; PSH[3] = 0; IMM[3] = 0;
POP[4] = 1; PSH[4] = 2; IMM[4] = 0;
POP[5] = 2; PSH[5] = 2; IMM[5] = 0;
POP[6] = 2; PSH[6] = 3; IMM[6] = 0;
POP[7] = 3; PSH[7] = 3; IMM[7] = 0;
for (let o = 0x10; o <= 0x16; o++) {
  POP[o] = 2; PSH[o] = 1; IMM[o] = 0;
}
POP[0x17] = 1; PSH[0x17] = 1; IMM[0x17] = 0;
for (let o = 0x18; o <= 0x1a; o++) {
  POP[o] = 2; PSH[o] = 1; IMM[o] = 0;
}
POP[0x1b] = 1; PSH[0x1b] = 1; IMM[0x1b] = 0;
for (let o = 0x1c; o <= 0x1e; o++) {
  POP[o] = 2; PSH[o] = 1; IMM[o] = 0;
}
for (let o = 0x20; o <= 0x22; o++) {
  POP[o] = 2; PSH[o] = 1; IMM[o] = 0;
}
POP[0x30] = 0; PSH[0x30] = 0; IMM[0x30] = 2;
POP[0x31] = 1; PSH[0x31] = 0; IMM[0x31] = 2;
POP[0x32] = 1; PSH[0x32] = 0; IMM[0x32] = 2;
POP[0x33] = 0; PSH[0x33] = 0; IMM[0x33] = 2;
POP[0x34] = 0; PSH[0x34] = 0; IMM[0x34] = 0;
POP[0x40] = 1; PSH[0x40] = 1; IMM[0x40] = 0;
POP[0x41] = 2; PSH[0x41] = 0; IMM[0x41] = 0;
POP[0x50] = 1; PSH[0x50] = 0; IMM[0x50] = 0;
POP[0x51] = 1; PSH[0x51] = 0; IMM[0x51] = 0;
POP[0x52] = 0; PSH[0x52] = 1; IMM[0x52] = 0;
POP[0x53] = 1; PSH[0x53] = 0; IMM[0x53] = 0;

function run(p: Uint8Array, s: Uint8Array): [Uint8Array, number] {
  const D: number[] = [];
  const C: number[] = [];
  const M: number[] = [];
  for (let i = 0; i < 256; i++) M.push(0);
  const out: number[] = [];
  let pc = 0;
  let ip = 0;
  let k = 0;
  let a = 0;
  let b = 0;
  let r = 0;
  const end = (c: number): [Uint8Array, number] => {
    return [Uint8Array.from(out), c];
  };
  const top = () => D[D.length - 1];
  for (;;) {
    if (k === 1000000) return end(10);
    if (pc < 0 || pc >= p.length) return end(9);
    const op = p[pc];
    if (POP[op] === undefined) return end(5);
    const im = IMM[op];
    if (pc + im >= p.length) return end(9);
    const po = POP[op];
    const pu = PSH[op];
    if (D.length < po) return end(2);
    if (D.length - po + pu > 256) return end(3);
    k++;
    pc += 1 + im;
    a = D.length > po ? D[D.length - po - 1] : 0;
    b = po ? D[D.length - 1] : 0;
    if (op === 0x00) return end(0);
    if (op === 0x01) { D.push(p[pc - 1] << 24 >> 24); continue; }
    if (op === 0x02) {
      r = p[pc - 4] | p[pc - 3] << 8 | p[pc - 2] << 16 | p[pc - 1] << 24;
     	D.push(r); continue;
    }
    if (op === 0x03) { D.pop(); continue; }
    if (op === 0x04) { D.push(D[D.length - 1]); continue; }
    if (op === 0x05) {
      const t = D[D.length - 1];
      D[D.length - 1] = D[D.length - 2];
      D[D.length - 2] = t;
      continue;
    }
    if (op === 0x06) {
      D.push(D[D.length - 2]);
      continue;
    }
    if (op === 0x07) {
      const c = D.pop()!;
      const y = D.pop()!;
      const z = D.pop()!;
      D.push(y, c, z);
      continue;
    }
    if ((op >= 0x10 && op <= 0x16) || (op >= 0x18 && op <= 0x1a)
      || (op >= 0x1c && op <= 0x1e) || (op >= 0x20 && op <= 0x22)) {
      const y = D.pop()! | 0;
      const z = D.pop()! | 0;
      const v = z >>> 0;
      const w = y >>> 0;
      if (op === 0x10) D.push((z + y) | 0);
      if (op === 0x11) D.push((z - y) | 0);
      if (op === 0x12) D.push(Math.imul(z, y));
      if (op === 0x13) {
        if (y === 0) return end(4);
        D.push(y === -1 ? (0 - z | 0) : (z / y | 0));
      }
      if (op === 0x14) {
        if (y === 0) return end(4);
        D.push(y === -1 || y === 1 ? 0 : (z % y) | 0);
      }
      if (op === 0x15) {
        if (y === 0) return end(4);
        D.push(v / w | 0);
      }
      if (op === 0x16) {
        if (y === 0) return end(4);
        D.push(v % w | 0);
      }
      if (op === 0x18) D.push(z & y);
      if (op === 0x19) D.push(z | y);
      if (op === 0x1a) D.push(z ^ y);
      if (op === 0x1c) D.push(z << (y & 31));
      if (op === 0x1d) D.push(z >> (y & 31));
      if (op === 0x1e) D.push(v >>> (y & 31) | 0);
      if (op === 0x20) D.push(z === y ? 1 : 0);
      if (op === 0x21) D.push(z < y ? 1 : 0);
      if (op === 0x22) D.push(v < w ? 1 : 0);
      continue;
    }
    if (op === 0x17) { D.push(0 - D.pop()! | 0); continue; }
    if (op === 0x1b) { D.push(~D.pop()!); continue; }
    if (op === 0x30 || op === 0x31 || op === 0x32) {
      r = (p[pc - 2] | p[pc - 1] << 8) << 16 >> 16;
      if (op === 0x31 && D.pop() !== 0) continue;
      if (op === 0x32 && D.pop() === 0) continue;
      pc += r;
      continue;
    }
    if (op === 0x33) {
      if (C.length === 64) return end(7);
      C.push(pc);
      pc = p[pc - 2] | p[pc - 1] << 8;
      continue;
    }
    if (op === 0x34) {
      if (C.length === 0) return end(8);
      pc = C.pop()!;
      continue;
    }
    if (op === 0x40 || op === 0x41) {
      a = D.pop()!;
      if (a >>> 0 > 255) return end(6);
      if (op === 0x40) D.push(M[a] ?? 0);
      else M[a] = D.pop()! | 0;
      continue;
    }
    if (op === 0x50) {
      r = String(D.pop()!);
      if (out.length + r.length > 65536) return end(11);
      for (let i = 0; i < r.length; i++) out.push(r.charCodeAt(i));
      continue;
    }
    if (op === 0x51) {
      if (out.length === 65536) return end(11);
      out.push(D.pop()! & 255);
      continue;
    }
    if (op === 0x52) {
      D.push(ip < s.length ? s[ip++] : -1);
      continue;
    }
    if (op === 0x53) return end(D.pop()! & 255);
  }
}

const I = process.env.MAKEFUN_INPUT_DIR ?? '/input';
const O = process.env.MAKEFUN_OUTPUT_DIR ?? '/output';
for (const id of fs.readdirSync(I).sort()) {
  const p = new Uint8Array(fs.readFileSync(I + '/' + id + '/program.bin'));
  const s = new Uint8Array(fs.readFileSync(I + '/' + id + '/input.bin'));
  const [o, c] = run(p, s);
  fs.mkdirSync(O + '/' + id, { recursive: true });
  fs.writeFileSync(O + '/' + id + '/stdout.bin', o);
  fs.writeFileSync(O + '/' + id + '/exit.txt', String(c) + '\n');
}
