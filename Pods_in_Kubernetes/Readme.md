# Kubernetes Foundations: Pods, Architecture, and Resource Management

This repository contains core concepts, architectural breakdowns, and practical commands for understanding Kubernetes Pods, distinguishing them from containers, mastering resource creation workflows (Imperative vs. Declarative), and inspecting active workloads. 

This material is designed to align directly with the practical, hands-on requirements of the **Certified Kubernetes Administrator (CKA)** exam.

---

## 📋 Table of Contents
1. [What is a Pod?](#1-what-is-a-pod)
2. [Containers vs. Pods](#2-containers-vs-pods)
3. [Imperative vs. Declarative Management](#3-imperative-vs-declarative-management)
4. [Hands-on: Imperative Pod Creation](#4-hands-on-imperative-pod-creation)
5. [Hands-on: Declarative Pod Creation](#5-hands-on-declarative-pod-creation)
6. [Deep-Dive Workload Inspection](#6-deep-dive-workload-inspection)
7. [CKA Exam Quick-Reference Cheat Sheet](#7-cka-exam-quick-reference-cheat-sheet)

---

## 1. What is a Pod?

A **Pod** is the smallest deployable and manageable unit of computing that you can create in Kubernetes. Rather than deploying containers directly, Kubernetes wraps one or more closely coupled containers into a single operational unit called a Pod.