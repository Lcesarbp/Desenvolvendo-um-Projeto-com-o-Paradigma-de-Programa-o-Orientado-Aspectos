# Desenvolvendo-um-Projeto-com-o-Paradigma-de-Programa-o-Orientado-Aspectos
Desafio da Digital Innovation one - Curso de Paradigma de Programação
Desafio de Projeto: Desenvolvendo um Projeto com o Paradigma de Programação Orientado à Aspectos


Neste projeto temos como proposta a implementação de uma solução envolvendo o Paradigma de Programação Orientado a Aspectos. Para que isso seja possível, é aconselhável trabalhar com um paradigma de programação primário para o projeto principal, como por exemplo, o paradigma Orientado a Objetos.

A proposta envolve criar uma função transversal ao código sendo aplicada em diferentes chamadas, no entanto, todas para o mesmo objetivo. Como exemplo de projeto de base podemos utilizar o seguinte problema:

Problema padrão: Em um sistema bancário, o cliente pode optar por fazer saques em diferentes contas (corrente, salário, poupança, investimento), no entanto, uma mensagem deve ser gerada informando caso o saldo seja insuficiente perante o valor requisitado. Essa mensagem deve ser gerada por meio de um log de erro envolvendo todas as contas, ou seja, todas as contas devem ser verificadas antes de liberar o dinheiro, analisando a disponibilidade do mesmo e, caso não seja possível, uma mensagem de saldo insuficiente deve ser gerada.

O programa principal implementando em Paradigma Orientado a Objetos deve abordar as funções básicas da conta bancária, já a verificação de saldo, por estar presente em todas a contas, deve ser implementado em Paradigma Orientado a Aspectos. Pode ser utilizada a linguagem Java com extensão da linguagem para aspectos “AspectJ”.

######SOLUÇÃO##########

//Criando uma classe base contabancaria

public abstract class ContaBancaria {
    protected double saldo;
    protected String tipoConta;

    public ContaBancaria(double saldoInicial, String tipoConta) {
        this.saldo = saldoInicial;
        this.tipoConta = tipoConta;
    }

    public double getSaldo() {
        return saldo;
    }

    public String getTipoConta() {
        return tipoConta;
    }

    public boolean podeSacar(double valor) {
        return saldo >= valor;
    }

    public boolean sacar(double valor) {
        if (podeSacar(valor)) {
            saldo -= valor;
            return true;
        }
        return false;
    }
}

//Criando as Subclasses de conta

public class ContaCorrente extends ContaBancaria {
    public ContaCorrente(double saldoInicial) {
        super(saldoInicial, "Conta Corrente");
    }
}

public class ContaPoupanca extends ContaBancaria {
    public ContaPoupanca(double saldoInicial) {
        super(saldoInicial, "Conta Poupança");
    }
}

public class ContaSalario extends ContaBancaria {
    public ContaSalario(double saldoInicial) {
        super(saldoInicial, "Conta Salário");
    }
}

public class ContaInvestimento extends ContaBancaria {
    public ContaInvestimento(double saldoInicial) {
        super(saldoInicial, "Conta Investimento");
    }
}

//implementando logica de saque

import java.util.List;

public class Cliente {
    private String nome;
    private List<ContaBancaria> contas;

    public Cliente(String nome, List<ContaBancaria> contas) {
        this.nome = nome;
        this.contas = contas;
    }

    public boolean sacar(double valor) {
        for (ContaBancaria conta : contas) {
            if (conta.podeSacar(valor)) {
                return conta.sacar(valor);
            }
        }
        return false;
    }

    public List<ContaBancaria> getContas() {
        return contas;
    }
}
//Criando o aspecto verificador de conta

import org.aspectj.lang.annotation.*;

@Aspect
public class VerificadorDeSaldo {

    @Before("execution(* Cliente.sacar(..)) && args(valor)")
    public void antesDeSacar(double valor) {
        System.out.println("[LOG] Cliente tentou sacar R$ " + valor);
    }

    @AfterReturning(pointcut = "execution(* Cliente.sacar(..))", returning = "sucesso")
    public void aposSaque(boolean sucesso) {
        if (!sucesso) {
            System.out.println("[ERRO] Nenhuma conta possui saldo suficiente para o saque.");
        } else {
            System.out.println("[SUCESSO] Saque realizado com sucesso!");
        }
    }

    @After("execution(* ContaBancaria.sacar(..)) && args(valor) && !execution(* Cliente.sacar(..))")
    public void logTentativaSaque(double valor) {
        System.out.println("[LOG] Tentativa de saque de R$ " + valor);
    }

    @AfterReturning(pointcut = "execution(* ContaBancaria.sacar(..))", returning = "sucesso")
    public void logResultadoSaque(boolean sucesso) {
        if (!sucesso) {
            System.out.println("[ERRO] Saldo insuficiente nesta conta.");
        }
    }
}
\\Classe principal sistemabancario

import java.util.Arrays;

public class SistemaBancario {
    public static void main(String[] args) {
        ContaBancaria conta1 = new ContaCorrente(300);
        ContaBancaria conta2 = new ContaPoupanca(200);
        ContaBancaria conta3 = new ContaSalario(100);
        ContaBancaria conta4 = new ContaInvestimento(500);

        Cliente cliente = new Cliente("João", Arrays.asList(conta1, conta2, conta3, conta4));

        System.out.println("\n--- Tentativa de saque de R$ 600 ---");
        cliente.sacar(600);

        System.out.println("\n--- Tentativa de saque de R$ 50 ---");
        cliente.sacar(50);

        System.out.println("\n--- Tentativa de saque de R$ 700 ---");
        cliente.sacar(700);
    }
}



