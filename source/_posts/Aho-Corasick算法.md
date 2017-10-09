---
title: Aho-Corasick算法
date: 2017-10-09 17:11:18
tags:
categories: algorithm
---
### Aho-Corasick算法
算法的关键是构造goto函数，fail函数和output函数，output函数的生成是伴随goto函数和fail函数的生成而生成。
##### goto function
```
Input: Set of keyword K={y1, y2,...., y3}.
Output: Goto function g and a partially computed output function output
Method: We assume output(s) is empty when state s is first created, and g(s, a) = fail if a is undefined or if g(s, a) has not yet been defined. The procedure enter(y) inserts into the goto graph a path that spells out y.

begin
    newstate<-0
    for i<-1 until k do enter(yi)
    for all a such that g(0, a) = fail do g(0, a)<-0
end

procedure enter(a1a2...am)
begin
    state<-0; j<-1
    while g(state, aj) != fail do
        begin
            state<-g(state, aj)
            j<-j+1
        end
    for p<-j until m do
    begin
        newstate<-newstate+1
        g(state, ap)<-newstate
        state<-newstate
    end
    output(state)<-{a1a2...am}
end
```
##### failure function
```
Input: Goto function g and output function output from up algorithm
Output: Failure function f and output function output
Method:
begin
    queue<-empty
    for each a such that g(0, a) = s != 0 do
        begin
            queue<-queue U {s}
            f(s)<-0
        end
    while queue != empty do
    begin
        let r be the next state in queue
        queue<-queue - {r}
        for each a such that g(r, a) = s != fail do
        begin
            queue<-queue U {s}
            state<-f(r)
            while g(state, a) = fail do state<-f(state)
            f(s)<-g(state, a)
            output(s)<-output(s) U output(f(s))
        end
    end
end
```
##### DFA
```
Input: Goto function g and failure function f
Output: Next move function o
Method:
begin
    queue<-empty
    for each symbol a do
        begin
            o(0, a)<-g(0, a)
            if g(0, a) != 0 then queue<-queue U {g(0, a)}
        end
    while queue != empty do
        begin
            let r be the next state in queue
            queue<-queue - {r}
            for each symbol a do
                if g(r, a) = s != fail do
                    begin
                        queue<-queue U {s}
                        o(r, a) <- s
                    end
                else
                    o(r, a)<-o(f(r), a)
        end
end
```