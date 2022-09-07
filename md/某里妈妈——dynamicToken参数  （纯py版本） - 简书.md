> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.jianshu.com](https://www.jianshu.com/p/b19ff8c84187)

原始代码，直接扣出来就能用。  
原始代码，翻译为纯py版本在下边

```
var xxx;
!function(require, exports, module) {
    !function() {
        function jsvm_this_initialization(D) {
            var h = 5;
            E: for (; h !== void 0; )
                switch (h % 10) {
                case 0:
                    (function(t, l) {
                        switch (l) {
                        case 0:
                            {
                                h = E < 64 ? 70 : 3;
                                return
                            }
                        case 1:
                            {
                                h = E < c ? 6 : 90;
                                return
                            }
                        case 2:
                            {
                                h = E < j ? 31 : 2;
                                return
                            }
                        case 3:
                            {
                                H = -H,
                                h = 51;
                                return
                            }
                        case 4:
                            {
                                E += 4,
                                h = 60;
                                return
                            }
                        case 5:
                            {
                                i += D.charCodeAt(E),
                                h = 1;
                                return
                            }
                        case 6:
                            {
                                h = E < V.length ? 9 : 12;
                                return
                            }
                        case 7:
                            {
                                o[P.charAt(E)] = E,
                                h = 91;
                                return
                            }
                        case 8:
                            {
                                h = H == 0 ? 41 : 51;
                                return
                            }
                        case 9:
                            {
                                E = 0,
                                h = 60;
                                return
                            }
                        }
                    }
                    )(arguments, h / 10 | 0);
                    break;
                case 1:
                    (function(t, l) {
                        switch (l) {
                        case 0:
                            {
                                E++,
                                h = 11;
                                return
                            }
                        case 1:
                            {
                                h = E < j ? 50 : 7;
                                return
                            }
                        case 2:
                            {
                                E++,
                                h = 20;
                                return
                            }
                        case 3:
                            {
                                H = H * 31 + ~E >>> 0,
                                k[E] = H % j,
                                h = 21;
                                return
                            }
                        case 4:
                            {
                                H = 13,
                                h = 51;
                                return
                            }
                        case 5:
                            {
                                E = 0,
                                h = 20;
                                return
                            }
                        case 6:
                            {
                                E += 4,
                                h = 10;
                                return
                            }
                        case 7:
                            {
                                E++,
                                h = 81;
                                return
                            }
                        case 8:
                            {
                                h = E < j ? 8 : 4;
                                return
                            }
                        case 9:
                            {
                                E++,
                                h = 0;
                                return
                            }
                        }
                    }
                    )(arguments, h / 10 | 0);
                    break;
                case 2:
                    (function(t, l) {
                        switch (l) {
                        case 0:
                            {
                                E = 0,
                                h = 81;
                                return
                            }
                        case 1:
                            {
                                jsvm_this_sdata = m.join("|"),
                                h = void 0;
                                break
                            }
                        }
                    }
                    )(arguments, h / 10 | 0);
                    break;
                case 3:
                    {
                        var T = m.pop();
                        c = T.length,
                        jsvm_this_insns = [],
                        E = 0,
                        h = 10;
                        break
                    }
                case 4:
                    {
                        D = m.join(""),
                        m = D.split("|");
                        var V = m.pop()
                          , P = m.pop()
                          , o = {};
                        E = 0,
                        h = 0;
                        break
                    }
                case 5:
                    {
                        var m = D.split(""), j = m.length, E, c, k = [], i = 0;
                        E = 0,
                        h = 11;
                        break
                    }
                case 6:
                    {
                        var J = o[T.charAt(E + 0)] << 18 | o[T.charAt(E + 1)] << 12 | o[T.charAt(E + 2)] << 6 | o[T.charAt(E + 3)];
                        jsvm_this_insns.push(J),
                        h = 61;
                        break
                    }
                case 7:
                    {
                        var H = ~(i * j);
                        h = H < 0 ? 30 : 80;
                        break
                    }
                case 8:
                    {
                        var M = k[E]
                          , I = m[M];
                        m[M] = m[0],
                        m[0] = I,
                        h = 71;
                        break
                    }
                case 9:
                    {
                        var J = o[V.charAt(E + 0)] << 18 | o[V.charAt(E + 1)] << 12 | o[V.charAt(E + 2)] << 6 | o[V.charAt(E + 3)];
                        jsvm_this_entrances.push(J),
                        h = 40;
                        break
                    }
                }
        }
        function jsvm_this_run(D, h) {
            function T(v, G) {
                var F = 1;
                E: for (; F !== void 0; )
                    switch (F % 3) {
                    case 0:
                        (function(S, Y) {
                            switch (Y) {
                            case 0:
                                {
                                    F = Q.length < G ? 3 : 2;
                                    return
                                }
                            case 1:
                                {
                                    Q = "0" + Q,
                                    F = 0;
                                    return
                                }
                            }
                        }
                        )(arguments, F / 3 | 0);
                        break;
                    case 1:
                        {
                            var Q = (+v).toString(16);
                            G = G || 2,
                            F = 0;
                            break
                        }
                    case 2:
                        return Q
                    }
            }
            function V(v) {
                return m[v]
            }
            function P(v, G) {
                m[v] = G
            }
            var o = 3;
            E: for (; o !== void 0; )
                switch (o % 7) {
                case 0:
                    (function(v, G) {
                        switch (G) {
                        case 0:
                            {
                                o = 35;
                                return
                            }
                        case 1:
                            {
                                l = !1,
                                o = t > jsvm_this_insns.length ? 2 : 6;
                                return
                            }
                        case 2:
                            {
                                t += f + 1,
                                o = 28;
                                return
                            }
                        case 3:
                            {
                                o = t === void 0 ? 1 : 15;
                                return
                            }
                        case 4:
                            {
                                o = 7;
                                return
                            }
                        case 5:
                            {
                                o = void 0;
                                break
                            }
                        case 6:
                            {
                                j = jsvm_this_entrances[h],
                                E = [],
                                c = [void 0],
                                k = [],
                                o = 8;
                                return
                            }
                        }
                    }
                    )(arguments, o / 7 | 0);
                    break;
                case 1:
                    (function(v, G) {
                        switch (G) {
                        case 0:
                            {
                                o = 35;
                                return
                            }
                        case 1:
                            {
                                o = 5;
                                return
                            }
                        case 2:
                            {
                                o = l === !1 ? 14 : 28;
                                return
                            }
                        case 3:
                            {
                                m = jsvm_this_sdata.split(" "),
                                i = 0,
                                o = 29;
                                return
                            }
                        case 4:
                            {
                                o = i < m.length ? 4 : 42;
                                return
                            }
                        case 5:
                            {
                                i++,
                                o = 29;
                                return
                            }
                        }
                    }
                    )(arguments, o / 7 | 0);
                    break;
                case 2:
                    return;
                case 3:
                    {
                        var m, j, E, c, k, i, J, H, M, I;
                        o = 22;
                        break
                    }
                case 4:
                    {
                        try {
                            m[i] = D(m[i])
                        } catch (v) {
                            m[i] = void 0
                        }
                        o = 36;
                        break
                    }
                case 5:
                    {
                        var t, l, y, p, u = 0, w = 0, Z = [], W = [], N = !0;
                        p = [void 0],
                        t = j - 1,
                        y = 0,
                        o = 7;
                        break
                    }
                case 6:
                    {
                        var X, e, n, a, s, r, _, g, R, O, f = 0;
                        switch (e = jsvm_this_insns[t],
                        X = e & 127,
                        X) {
                        case 26:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            r = n >> 0 & 31,
                            E[a][s] = E[r];
                            break;
                        case 21:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] === E[r];
                            break;
                        case 64:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = V(E[s] + E[r]);
                            break;
                        case 96:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 127,
                            r |= (n >> 0 & 511) << 7,
                            E[a] = V(E[s] + r);
                            break;
                        case 16:
                            a = e >> 7 & 31,
                            s = e >> 12 & 255,
                            E[a] = s == 2 ? +E[a] : s == 0 ? {} : s == 1 ? [] : void 0;
                            break;
                        case 32:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            s |= (n >> 0 & 15) << 12,
                            E[a] = s;
                            break;
                        case 48:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            s |= (n >> 0 & 15) << 12,
                            s = s << 16 >> 16,
                            E[a] = s;
                            break;
                        case 80:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 65535,
                            s = e >> 23 & 1,
                            s |= (n >> 0 & 15) << 1,
                            r = n >> 4 & 65535,
                            P(a + r, E[s]);
                            break;
                        case 8:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            _ = e >> 22 & 3,
                            _ |= (n >> 0 & 7) << 2,
                            g = n >> 3 & 31;
                            try {
                                s === 31 ? E[a] = E[r](E[_], E[g]) : E[a] = E[s][E[r]](E[_], E[g])
                            } catch (v) {
                                if (l = !0,
                                t = p.pop(),
                                t == null)
                                    break;
                                t === -1 && (t = p.pop()),
                                u === 2 && (u = p.pop(),
                                t = p.pop(),
                                t === -1 && (c.pop(),
                                t = p.pop())),
                                w = 3 + u,
                                u = (u + 1) % 3,
                                E[0] = v
                            }
                            break;
                        case 112:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            s |= (n >> 0 & 15) << 12,
                            r = n >> 4 & 31,
                            E[a] = V(s + E[r]);
                            break;
                        case 72:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            _ = e >> 22 & 3,
                            _ |= (n >> 0 & 7) << 2;
                            try {
                                s === 31 ? E[a] = E[r](E[_]) : E[a] = E[s][E[r]](E[_])
                            } catch (v) {
                                if (l = !0,
                                t = p.pop(),
                                t == null)
                                    break;
                                t === -1 && (t = p.pop()),
                                u === 2 && (u = p.pop(),
                                t = p.pop(),
                                t === -1 && (c.pop(),
                                t = p.pop())),
                                w = 3 + u,
                                u = (u + 1) % 3,
                                E[0] = v
                            }
                            break;
                        case 104:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            _ = e >> 22 & 3,
                            _ |= (n >> 0 & 7) << 2,
                            g = n >> 3 & 31,
                            R = n >> 8 & 31,
                            O = n >> 13 & 31;
                            try {
                                if (y === 0)
                                    s === 31 ? E[a] = E[r](E[_], E[g], E[R], E[O]) : E[a] = E[s][E[r]](E[_], E[g], E[R], E[O]);
                                else {
                                    for (H = [],
                                    s == 31 ? M = void 0 : M = E[s],
                                    H.push(E[_]),
                                    H.push(E[g]),
                                    H.push(E[R]),
                                    H.push(E[O]),
                                    I = [],
                                    i = 0; i < y; i++)
                                        I.push(c.pop());
                                    for (i = 0; i < y; i++)
                                        H.push(I.pop());
                                    s == 31 ? E[a] = E[r].apply(M, H) : E[a] = M[E[r]].apply(M, H),
                                    y = 0
                                }
                            } catch (v) {
                                if (l = !0,
                                t = p.pop(),
                                t == null)
                                    break;
                                t === -1 && (t = p.pop()),
                                u === 2 && (u = p.pop(),
                                t = p.pop(),
                                t === -1 && (c.pop(),
                                t = p.pop())),
                                w = 3 + u,
                                u = (u + 1) % 3,
                                E[0] = v
                            }
                            break;
                        case 24:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31;
                            try {
                                s === 31 ? E[a] = E[r]() : E[a] = E[s][E[r]]()
                            } catch (v) {
                                if (l = !0,
                                t = p.pop(),
                                t == null)
                                    break;
                                t === -1 && (t = p.pop()),
                                u === 2 && (u = p.pop(),
                                t = p.pop(),
                                t === -1 && (c.pop(),
                                t = p.pop())),
                                w = 3 + u,
                                u = (u + 1) % 3,
                                E[0] = v
                            }
                            break;
                        case 88:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            P(E[a] + E[r], E[s]);
                            break;
                        case 40:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            _ = e >> 22 & 3,
                            _ |= (n >> 0 & 7) << 2,
                            g = n >> 3 & 31,
                            R = n >> 8 & 31;
                            try {
                                s === 31 ? E[a] = E[r](E[_], E[g], E[R]) : E[a] = E[s][E[r]](E[_], E[g], E[R])
                            } catch (v) {
                                if (l = !0,
                                t = p.pop(),
                                t == null)
                                    break;
                                t === -1 && (t = p.pop()),
                                u === 2 && (u = p.pop(),
                                t = p.pop(),
                                t === -1 && (c.pop(),
                                t = p.pop())),
                                w = 3 + u,
                                u = (u + 1) % 3,
                                E[0] = v
                            }
                            break;
                        case 120:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            E[a] = D(E[s]);
                            break;
                        case 4:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            y += 3,
                            c.push(E[a]),
                            c.push(E[s]),
                            c.push(E[r]);
                            break;
                        case 68:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 127,
                            r |= (n >> 0 & 511) << 7,
                            P(E[a] + r, E[s]);
                            break;
                        case 36:
                            a = e >> 7 & 31,
                            y += 1,
                            c.push(E[a]);
                            break;
                        case 56:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            _ = e >> 22 & 3,
                            _ |= (n >> 0 & 7) << 2,
                            y += 4,
                            c.push(E[a]),
                            c.push(E[s]),
                            c.push(E[r]),
                            c.push(E[_]);
                            break;
                        case 100:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            y += 2,
                            c.push(E[a]),
                            c.push(E[s]);
                            break;
                        case 20:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 65535,
                            s = e >> 23 & 1,
                            s |= (n >> 0 & 15) << 1,
                            r = n >> 4 & 31,
                            P(a + E[r], E[s]);
                            break;
                        case 84:
                            a = e >> 7 & 65535,
                            a = a << 16 >> 16,
                            p.push(t + a);
                            break;
                        case 52:
                            a = e >> 7 & 65535,
                            a = a << 16 >> 16,
                            p.push(t + a);
                            break;
                        case 116:
                            a = e >> 7 & 65535,
                            a = a << 16 >> 16,
                            p.push(u),
                            w = 0,
                            u = 0,
                            p.push(t + a);
                            break;
                        case 12:
                            a = e >> 7 & 31,
                            E[0] = E[a],
                            w = 3 + u;
                            break;
                        case 76:
                            l = !0,
                            t = p.pop(),
                            u = p.pop(),
                            w > 3 && (t = p.pop(),
                            t === -1 && (c.pop(),
                            t = p.pop())),
                            w = 0;
                            break;
                        case 44:
                            l = !0,
                            t = p.pop(),
                            u++,
                            w === 0 && (t = p.pop(),
                            u++);
                            break;
                        case 108:
                            l = !0,
                            t = p.pop(),
                            u++;
                            break;
                        case 92:
                            a = e >> 7 & 65535,
                            l = !0,
                            t = a - 1;
                            break;
                        case 60:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            s |= (n >> 0 & 15) << 12,
                            E[a] = s;
                            break;
                        case 124:
                            a = e >> 7 & 31,
                            l = !0,
                            t = E[a] - 1;
                            break;
                        case 28:
                            a = e >> 7 & 31,
                            l = !0,
                            N = !1,
                            c.push(t + 1 + f),
                            t = E[a] - 1,
                            p.push(-1),
                            u = 0,
                            w = 0;
                            break;
                        case 66:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            jsvm_this_tmpValue = E[s],
                            D("" + E[a] + " = jsvm_this_tmpValue;");
                            break;
                        case 2:
                            a = e >> 7 & 31,
                            E[a] = {};
                            break;
                        case 98:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] | E[r];
                            break;
                        case 34:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] ^ E[r];
                            break;
                        case 82:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] % E[r];
                            break;
                        case 18:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] / E[r];
                            break;
                        case 114:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] & E[r];
                            break;
                        case 50:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            E[a] = ~E[s];
                            break;
                        case 74:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] * E[r];
                            break;
                        case 10:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] - E[r];
                            break;
                        case 42:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] + E[r];
                            break;
                        case 106:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] >>> E[r];
                            break;
                        case 0:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            s |= (n >> 0 & 15) << 12,
                            r = n >> 4 & 65535,
                            E[a] = V(s + r);
                            break;
                        case 58:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] << E[r];
                            break;
                        case 90:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 127,
                            r |= (n >> 0 & 31) << 7,
                            E[a] = E[s][r];
                            break;
                        case 6:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            s |= (n >> 0 & 15) << 12,
                            E[a] && (l = !0,
                            t = s - 1);
                            break;
                        case 70:
                            l = !0,
                            p.pop(),
                            t = c.pop(),
                            t === void 0 && (t = -1);
                            break;
                        case 122:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            E[a] = D("" + E[s]);
                            break;
                        case 38:
                            if (f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 65535,
                            a = a << 16 >> 16,
                            s = e >> 23 & 1,
                            s |= (n >> 0 & 15) << 1,
                            c.length <= a)
                                break;
                            c[c.length - 1 - a] = E[s];
                            break;
                        case 22:
                            if (a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            c.length <= E[s])
                                break;
                            E[a] = c[c.length - 1 - E[s]];
                            break;
                        case 86:
                            if (a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            c.length <= E[a])
                                break;
                            c[c.length - 1 - E[a]] = E[s];
                            break;
                        case 54:
                            if (a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] === void 0)
                                jsvm_this_tmpValue = E[r],
                                D("" + E[s] + " = jsvm_this_tmpValue;");
                            else
                                try {
                                    E[a][E[s]] = E[r]
                                } catch (v) {
                                    if (l = !0,
                                    t = p.pop(),
                                    t == null)
                                        break;
                                    t === -1 && (t = p.pop()),
                                    u === 2 && (u = p.pop(),
                                    t = p.pop(),
                                    t === -1 && (c.pop(),
                                    t = p.pop())),
                                    w = 3 + u,
                                    u = (u + 1) % 3,
                                    E[0] = v
                                }
                            break;
                        case 102:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            E[a] && (l = !0,
                            t = E[s] - 1);
                            break;
                        case 118:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31;
                            try {
                                E[a] = E[s][E[r]]
                            } catch (v) {
                                if (l = !0,
                                t = p.pop(),
                                t == null)
                                    break;
                                t === -1 && (t = p.pop()),
                                u === 2 && (u = p.pop(),
                                t = p.pop(),
                                t === -1 && (c.pop(),
                                t = p.pop())),
                                w = 3 + u,
                                u = (u + 1) % 3,
                                E[0] = v
                            }
                            break;
                        case 14:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            _ = e >> 22 & 3,
                            _ |= (n >> 0 & 7) << 2,
                            c.push(E[a]),
                            c.push(E[s]),
                            c.push(E[r]),
                            c.push(E[_]);
                            break;
                        case 46:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = c.pop(),
                            k.push(E[a]),
                            E[s] = c.pop(),
                            k.push(E[s]),
                            E[r] = c.pop(),
                            k.push(E[r]);
                            break;
                        case 78:
                            a = e >> 7 & 31,
                            E[a] = c.pop(),
                            k.push(E[a]);
                            break;
                        case 30:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            E[a] && c.push(E[s]);
                            break;
                        case 94:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            c.push(E[a]),
                            c.push(E[s]),
                            c.push(E[r]);
                            break;
                        case 62:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            E[a] = E[s];
                            break;
                        case 126:
                            a = e >> 7 & 31,
                            c.push(E[a]);
                            break;
                        case 110:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            E[a] = c.pop(),
                            k.push(E[a]),
                            E[s] = c.pop(),
                            k.push(E[s]);
                            break;
                        case 65:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] >= E[r];
                            break;
                        case 1:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] <= E[r];
                            break;
                        case 33:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] < E[r];
                            break;
                        case 97:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] > E[r];
                            break;
                        case 17:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] && E[r];
                            break;
                        case 81:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] || E[r];
                            break;
                        case 113:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            E[a] = !E[s];
                            break;
                        case 9:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] !== E[r];
                            break;
                        case 49:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] >> E[r];
                            break;
                        case 41:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] != E[r];
                            break;
                        case 73:
                            if (f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            s |= (n >> 0 & 15) << 12,
                            s = s << 16 >> 16,
                            c.length <= s)
                                break;
                            E[a] = c[c.length - 1 - s];
                            break;
                        case 105:
                            f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            _ = e >> 22 & 3,
                            _ |= (n >> 0 & 7) << 2,
                            E[a] = c.pop(),
                            k.push(E[a]),
                            E[s] = c.pop(),
                            k.push(E[s]),
                            E[r] = c.pop(),
                            k.push(E[r]),
                            E[_] = c.pop(),
                            k.push(E[_]);
                            break;
                        case 89:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s]in E[r];
                            break;
                        case 57:
                            a = e >> 7 & 31,
                            E[a] = {};
                            break;
                        case 121:
                            if (f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 31,
                            s = e >> 12 & 4095,
                            s |= (n >> 0 & 15) << 12,
                            r = n >> 4 & 255,
                            typeof E[a].jsvmfunc == "number") {
                                for (i = 1; i <= r; i++)
                                    W.push(V(s + i));
                                N = !0,
                                c.push(V(s)),
                                l = !0,
                                c.push(t + 1 + f),
                                t = E[a].jsvmfunc - 1,
                                p.push(-1),
                                u = 0,
                                w = 0
                            } else {
                                for (H = [],
                                M = V(s),
                                i = 0; i < r; i++)
                                    H.push(V(s + r - i));
                                typeof E[a] == "function" && c.push(E[a].apply(M, H))
                            }
                            break;
                        case 25:
                            a = e >> 7 & 31,
                            E[a] = [];
                            break;
                        case 69:
                            if (a = e >> 7 & 255,
                            N)
                                for (i = 0; i < a; i++)
                                    c.push(W.pop());
                            W = [],
                            N = !1;
                            break;
                        case 37:
                            for (f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 65535,
                            s = e >> 23 & 1,
                            s |= (n >> 0 & 32767) << 1,
                            i = 1; i <= s; i++)
                                P(a + s - i, c.pop());
                            break;
                        case 101:
                            a = e >> 7 & 31,
                            s = e >> 12 & 31,
                            r = e >> 17 & 31,
                            E[a] = E[s] == E[r];
                            break;
                        case 5:
                            for (f = 1,
                            n = jsvm_this_insns[t + 1],
                            a = e >> 7 & 65535,
                            s = e >> 23 & 1,
                            s |= (n >> 0 & 32767) << 1,
                            i = 0; i < s; i++)
                                c.push(V(a + i));
                            break;
                        default:
                            for (a = e >> 7 & 255,
                            i = 0; i < a; i++)
                                c.push(k.pop());
                            break
                        }
                        o = t === -1 ? 0 : 21;
                        break
                    }
                }
        }
        var jsvm_this_tmpValue, jsvm_this_insns = [], jsvm_this_sdata = [], jsvm_this_entrances = [], jsvm_this_privs = [];
        jsvm_this_initialization("HeE   GVE EHw''H'z'i'MEEE '1sEmEbspinVVetVbEHrEEaE'EDdcjie    EjsEEEEEE'HcNdtpiMgdvcndEE4nEd  uErewenEd'BneEEaEr  aEnEeE8aud  inEuEEEEnl4fEE/dEErEEtvuE''H7EEVirEd    HgElEtrEepeHEEetcvEc 'e tEiseG/cHEEHEeEEuHEnnmbuauAEr0wcMg ElUEdenE2E2  EerVEE  'sHlEt'aVEhaDC9deEEHE'leEgthE   uEd'fWqe'|EuElEEEepVEEEVE''EEEnEVHpEayEEEjE/ElEEEcEH    EDDEpuH'eEEEgEEHEhuVEEEEpEEED   EpEEUuEEEEpEa'cpu'EVEaEE    EEDEEEEHucEinDHEEHpnEEEEE0EnEcEEEEnVElMEEEeEGLoEuEyjGplEEaE4cEEnuEC1rEpEVHEEEEopHEEH4p+EVE'EEEgEErmEDVcElEEEEHEEHlEHE'EEEGED1EEErEEm8cErE2EEUEEElpEaEEEplEHEaHEEE7EEV'EELEEEEH'pVHEEEHEeEHEruEE7EHElEeglgHgErDEEEEaV42EEVDEKe2EEsV1EuEcEEupEwel4EEEEu_EEEEEEVEEVueEMEcHagEtEweHpE   peEE'EEEE'EEcEHDEEEEEt1E'1EEEbEbEEVE    rEHwlEtEEEEGEcdUcEhED1EEErEeEVeElE  EEDEEHEEEEV'EEMDEEuE'EEHEEEecEuEEn1GE9EEEEVbHIEcpEiagEEgEnuEEeEceEsdEE1EEEEMpE4EEEEVDuuwEHEeupacE11EuEEwE/acuEwEEElDHEgdE7GEEEEEEDEDEE14EEEEEEEeeEEaE   hcHM'HElEHoulwElEEpEEEEp7aEDEc  EaHEEuHEEVpBMl  lEEuEegccEEEcE4aEEucpEEcEEEEVEugEEEeEElDEHEMEEEuEEEMDLDEVEEuEwEt1ugcHgrHVEEEEErEEEpEiEEEEHEEEEEeEEEEEElcEEEVEu  eEEExoHEEEEudEEDEVEv8EcEE6  MaEEaUEE8HMlDH'EEHEEEaEEEDiEmHEEEciVEeEEEHEEacEEgHEEgEcEwiEEEiEuGEEOUEElpHEEd7Eu'EvEgfpeGEEHgwE'weEaenEeGEHEcEEEEeoEEeHEtEEhEVDeufElEu  EYVKBEEEE'EEEEEpEneEEEV1tEEEEEaEEEDEEaHnuEcEEEE'j'HEEEEEEEHEDEjiEv8eGEEEEu6VDEeEEaHHlERwe1EEEVeEHEDEaEEEaeREEEE 7rEEEafEbpEEEVaEu'cEHrEEfEeEaV4EEEHh    EHEGEEfEVHEEVppEa   iEEEE7EEEEVulFEaEEutEEEcEEDrEwEjsP1UEcEHclDdEEeEVfVplsEuHEuEEEHnEE1iluGErEEEEEEHCcGEE8aeEEVH    ctEEQD2EEwEEVEMpeEEcEE8EEEbEEEEEEdrEBVEEEEuccEHpEwE EE4'ugcpEEeeEED'EaHE        8lEEEEEupEEVEHEDHuEVVcXn2EcaHEEVcEElEEEcuEglLmEaHEnViEEEHpgEpE  a3ELExE7rEEEHSsgEEEEleEVe8DEvDDclpE7MwaEpEEEEDE7rEfEgMuVV2E7reEEE2EaMer1VH3uMDEHEEEEudEEnEMEcrEuHeEE'H  EV7EEEpEEVcwEE_'Euu4Egru1UHMfEsEEEEEEEEEVGeeElEUlaH VHCaEEMEEu8E1aEEEeHeEEr|DVDEEaE2KdEaEgtEHx4nA5DGJcE7yTEoEEjc'EEEfPEE8fcEnZUEk6EEsB28ESEq|EEEuErEA"),
        xxx = function(n, t, r) {
            n = n || "",
            t = t || 2;
            var e, o, s, u, i, jsvmportal_0_1 = function() {
                var inout = arguments, retval;
                return jsvm_this_run(function() {
                    return eval(arguments[0])
                }, 0),
                retval
            };
            return jsvm_this_run(function() {
                return eval(arguments[0])
            }, 1),
            "" + e(u) + e(i)
        }
    }
    ()
}();
console.log(xxx('164264751781901864590029', 3, '1642650295289')) 
```

jsvm 混淆的 翻译成py 后很简单。

```
from functools import reduce
class ParseDynamicToken:
    def baseN(self, num, b):
        '''
        进制转换
        :param num:
        :param b:
        :return:
        '''
        return ((num == 0) and "0") or (
                self.baseN(num // b, b).lstrip("0") + "0123456789abcdefghijklmnopqrstuvwxyz"[num % b])

    def parse_core(self, key):
        '''
        解密逻辑
        :param key:
        :return:
        '''
        return reduce(lambda x, y: str(x) + str(y), map(lambda x: ord(x) * 4, str(key)))

    def get_dynamicToken(self, seedToken: str, timestr: int):
        a1 = seedToken[3:11]
        a2 = self.baseN(timestr, 36)
        return self.parse_core(a1) + self.parse_core(a2) 
```