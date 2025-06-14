## t-TMD 一箩筐

### MODEL: 1-layer & 2-layer
K and K', k and k'
polarization

### CORRELATED INSULATOR

### TOPO



---
## parameters for twisted-homo-bilayer-TMD
> 让我们看看这个哈密顿量都能整出什么幺蛾子



- The original hamiltonian is for **electrons** [^1] 
$$ H=\left[
\begin{matrix}
-\frac{(\hat{k}-K_t)^2}{2m^*}+Δ_t(r) & Δ_T(r) \\
Δ_T^\dagger(r) & -\frac{(\hat{k}-K_b)^2}{2m^*}+Δ_b(r)
\end{matrix}
\right]
$$  
<font size=1> $m^*$ is the effective mass. $K_t,K_b$ is the $K$-point of top/bottom layer BZ, and $K_b-K_t=q_0^{(1)}$ </font>
<font size=1> $Δ_l(r)=
 2V_1 \sum_{1,3,5}{\cos(\mathbf{g}_i^{(1)}·\mathbf{r}+l ϕ_1)}
+2V_2 \sum_{1,3,5}{\cos(\mathbf{g}_i^{(2)}·\mathbf{r})};
\; l=\pm 1$ for top/bottom layer 
$Δ_T=
  w_1 \sum_{1,2,3}{e^{-{\rm i} \mathbf{q}_i^{(1)}·\mathbf{r} }} + 
  w_2 \sum_{1,2,3}{e^{-{\rm i} \mathbf{q}_i^{(2)}·\mathbf{r}}}
  $ </font>
<font size=1>$\mathbf{g_0^{(1)}}=
\frac{4\pi}{\sqrt{3}\,a_M} \hat{x}
\xrightarrow{C_6}                   \;
\mathbf{g_1^{(1)}}$ </font>
<font size=1>$\mathbf{g_0^{(2)}}=   \ \ 
\frac{4\pi}{a_M} \hat{y}            \ \ 
\xrightarrow{C_6}                   \;
\mathbf{g_1^{(2)}}$ </font>
<font size=1>$\mathbf{q_0^{(1)}}=   \ 
\frac{4\pi}{3\,a_M} \hat{y}         \  
\xrightarrow{C_3}                   \; 
\mathbf{q_1^{(1)}}$ </font>
<font size=1>$\mathbf{q_0^{(2)}}=   
-\frac{8\pi}{3\,a_M} \hat{y}         
\xrightarrow{C_3}                   \;
\mathbf{q_1^{(2)}}$ </font>
<br>


- For **holes**: diag → - diag, off-diag → off-diag* [^2]
<font size=2.4>
  - "creating a hole" is equivalent to "annihilating a electron", so a minus sign is added for the diagonal element.
  - "a hole hopping from top layer to bottom" is equivalent to "a electron hopping from bottom to top", so the hopping strength is the same, yet the phase differs by a dagger.
  - and often we switch the top and bottom layer, thus another dagger to the off-diagonal element, and cancels out. </font>  
$$ H=\left[
\begin{matrix}
    \frac{(\hat{k}-K_b)^2}{2m^*}-Δ_b(r) &       Δ_T(r)        \\
            Δ_T^\dagger(r)              & \frac{(\hat{k}-K_t)^2}{2m^*}-Δ_t(r)
\end{matrix}
\right]
$$ Note here we swap the top and bottom layer. 
<br>


- To keep the kinetic energy in the form of $\nabla^2$, a gauge transformation is performed
<font size=2.4>
  - the wavefunction is transformed by <font size=2>$\ |\psi_b\rang, |\psi_t\rang → {\rm e}^{-{\rm i} K_b r}|\psi_b\rang, {\rm e}^{-{\rm i} K_t r}|\psi_t\rang$ </font> </font>
$$ H=\left[
\begin{matrix}
    \frac{\hat{k}^2}{2m^*}-Δ_b(r) &       \tilde{Δ}_T(r)        \\
            \tilde{Δ}_T^\dagger(r)              & \frac{\hat{k}^2}{2m^*}-Δ_t(r)
\end{matrix}
\right] 
$$ <font size=1> $
\tilde{Δ}_T(r) := {\rm e}^{{\rm i}(-K_t+K_b)r } 
Δ_T (r)=
  w_1 ({ 1 + e^{-{\rm i}\mathbf{g}_4^{(1)}·\mathbf{r}} + e^{-{\rm i}\mathbf{g}_5^{(1)}·\mathbf{r}} }) + 
  w_2 \left({ e^{{\rm i} \mathbf{g}_0^{(2)}·\mathbf{r}} + 2\cos(g_0^{(1)}r) }\right)
  $ 
</font><br>

- We can also split the Hamiltonian into Pauli matrices
$$
H = \sum_{0,x,y,z}{h_i \, σ^i}
$$ <font size=1> $σ^0=𝟙$
$h_0 = \hat{k}^2/2m^* - 2 V_1 \left(\sum_{1,3,5}{\cos( \mathbf{g}_i^{(1)} · \mathbf{r} )} \right)\cos(\phi_1) - 2V_2 \left(\sum_{1,3,5}{\cos( \mathbf{g}_i^{(2)} · \mathbf{r} )} \right) $
$h_z = - 2 V_1 \left(\sum_{1,3,5}{\sin( \mathbf{g}_i^{(1)} · \mathbf{r} )} \right)\sin(\phi_1) $ 
$h_x = {\rm Re}{\tilde{Δ}_T}, h_y = -{\rm Im}{\tilde{Δ}_T}$</font>

tMoTe2 @ ~ 2.1°
[$ϕ_1=-90.0°, V_1=2.4\;{\rm meV}, V_2=1.0\;{\rm meV}, w_1=-5.8\;{\rm meV}, w_2=2.8\;{\rm meV}$, fitted at 1.89°, $m^*=0.6m_e, \epsilon=5$]



[^1]: See e.g., *Cheng Xu, Ning Mao, Tiansheng Zeng, Yang Zhang
Multiple Chern bands in twisted MoTe2 and possible non-Abelian states.
arXiv:2403.17003v2*
[^2]: See e.g., eq. 39 in *Aidan P. Reddy, Nisarga Paul, Ahmed Abouelkomsan, Liang Fu. Non-Abelian Fractionalization in Topological Minibands. Physical Review Letters 133, 166503 (2024). https://doi.org/10.1103/PhysRevLett.133.166503.*

