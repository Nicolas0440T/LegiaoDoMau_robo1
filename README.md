





















package legiaoDoMauT;

import robocode.*;
import robocode.util.Utils;
import java.awt.*;
import java.awt.geom.Point2D;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;

public class legiaoDoMau_robo1 extends TeamRobot {

    private boolean movendoParaFrente = true;
    private double ultimoEnergiaInimigo = 100;
    private final Map<String, Point2D.Double> posicoesAliados = new HashMap<>();

    public void run() {

        // ===== CORES ROXO + VERDE =====
        setBodyColor(new Color(128, 0, 128)); // Roxo
        setGunColor(Color.green);             // Verde
        setRadarColor(Color.green);           // Verde
        setBulletColor(Color.green);          // Verde
        setScanColor(Color.green);            // Verde
        // ==============================

        setAdjustGunForRobotTurn(true);
        setAdjustRadarForGunTurn(true);
        setAdjustRadarForRobotTurn(true);

        while (true) {
            try {
                broadcastMessage(new Point2D.Double(getX(), getY()));
            } catch (IOException ignored) {}

            setTurnRadarRight(360);
            evitarParedesConstante();
            execute();
        }
    }

    public void onScannedRobot(ScannedRobotEvent e) {

        if (isTeammate(e.getName()))
            return;

        double distancia = e.getDistance();
        double velocidadeInimigo = e.getVelocity();
        double anguloInimigo = e.getBearing();
        double energiaInimigo = e.getEnergy();

        double queda = ultimoEnergiaInimigo - energiaInimigo;
        if (queda > 0 && queda <= 3) {
            evasaoPorTiro(e);
        }
        ultimoEnergiaInimigo = energiaInimigo;

        double anguloRadar = getHeading() + anguloInimigo - getRadarHeading();
        setTurnRadarRight(Utils.normalRelativeAngleDegrees(anguloRadar));

        double anguloAbs = getHeading() + anguloInimigo;
        double potencia = escolherForca(distancia);
        double tempo = distancia / bulletSpeed(potencia);
        double deslocamento = velocidadeInimigo * tempo;
        double ajuste = Math.toDegrees(Math.atan2(deslocamento, Math.max(distancia, 1)));
        double direcaoTiro = anguloAbs + (velocidadeInimigo < 0 ? -ajuste : ajuste);
        double virarA
