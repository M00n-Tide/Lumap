// 风险分析实体类
import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "risk_analyzer")
class RiskAnalyzer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "chem_id")
    private Long chemId;
    private String riskType;
    private Integer riskLevel;
    @Column(name = "analysis_time")
    private LocalDateTime analysisTime;
    private String suggestion;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public Long getChemId() { return chemId; }
    public void setChemId(Long chemId) { this.chemId = chemId; }
    public String getRiskType() { return riskType; }
    public void setRiskType(String riskType) { this.riskType = riskType; }
    public Integer getRiskLevel() { return riskLevel; }
    public void setRiskLevel(Integer riskLevel) { this.riskLevel = riskLevel; }
    public LocalDateTime getAnalysisTime() { return analysisTime; }
    public void setAnalysisTime(LocalDateTime analysisTime) { this.analysisTime = analysisTime; }
    public String getSuggestion() { return suggestion; }
    public void setSuggestion(String suggestion) { this.suggestion = suggestion; }
}

// 风险监控实体类
@Entity
@Table(name = "risk_monitor")
class RiskMonitor {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "analyzer_id")
    private Long analyzerId;
    @Enumerated(EnumType.STRING)
    private MonitorStatus monitorStatus;
    @Column(name = "last_check_time")
    private LocalDateTime lastCheckTime;
    private String threshold;

    // 【已修复】新增 MonitorStatus 枚举
    public enum MonitorStatus { 运行中, 已停止, 告警 }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public Long getAnalyzerId() { return analyzerId; }
    public void setAnalyzerId(Long analyzerId) { this.analyzerId = analyzerId; }
    public MonitorStatus getMonitorStatus() { return monitorStatus; }
    public void setMonitorStatus(MonitorStatus monitorStatus) { this.monitorStatus = monitorStatus; }
    public LocalDateTime getLastCheckTime() { return lastCheckTime; }
    public void setLastCheckTime(LocalDateTime lastCheckTime) { this.lastCheckTime = lastCheckTime; }
    public String getThreshold() { return threshold; }
    public void setThreshold(String threshold) { this.threshold = threshold; }
}

// 数据访问接口
import org.springframework.data.jpa.repository.JpaRepository;
interface RiskAnalyzerRepository extends JpaRepository<RiskAnalyzer, Long> {
    List<RiskAnalyzer> findByChemId(Long chemId);
}

interface RiskMonitorRepository extends JpaRepository<RiskMonitor, Long> {
    List<RiskMonitor> findByMonitorStatus(RiskMonitor.MonitorStatus status);
}

// 服务类
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.time.LocalDateTime;
import java.util.List;

@Service
@Transactional
class RiskMonitorService {
    private final RiskAnalyzerRepository analyzerRepo;
    private final RiskMonitorRepository monitorRepo;

    public RiskMonitorService(RiskAnalyzerRepository analyzerRepo, RiskMonitorRepository monitorRepo) {
        this.analyzerRepo = analyzerRepo;
        this.monitorRepo = monitorRepo;
    }

    public RiskAnalyzer analyzeRisk(RiskAnalyzer analyzer) {
        analyzer.setAnalysisTime(LocalDateTime.now());
        return analyzerRepo.save(analyzer);
    }

    // 【已修复】使用枚举替代硬编码字符串
    public RiskMonitor startMonitor(Long analyzerId, String threshold) {
        RiskMonitor monitor = new RiskMonitor();
        monitor.setAnalyzerId(analyzerId);
        monitor.setMonitorStatus(RiskMonitor.MonitorStatus.运行中);
        monitor.setLastCheckTime(LocalDateTime.now());
        monitor.setThreshold(threshold);
        return monitorRepo.save(monitor);
    }

    public List<RiskMonitor> getActiveMonitors() {
        return monitorRepo.findByMonitorStatus(RiskMonitor.MonitorStatus.运行中);
    }
}

// 控制器
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/risk")
class RiskMonitorController {
    private final RiskMonitorService service;

    public RiskMonitorController(RiskMonitorService service) {
        this.service = service;
    }

    @PostMapping("/analyze")
    public RiskAnalyzer analyze(@RequestBody RiskAnalyzer analyzer) {
        return service.analyzeRisk(analyzer);
    }

    @PostMapping("/monitor/start")
    public RiskMonitor startMonitor(@RequestParam Long analyzerId, @RequestParam String threshold) {
        return service.startMonitor(analyzerId, threshold);
    }

    @GetMapping("/monitor/active")
    public List<RiskMonitor> getActive() {
        return service.getActiveMonitors();
    }
}