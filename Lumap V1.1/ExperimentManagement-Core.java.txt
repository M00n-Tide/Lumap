// 实验方案实体类
import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "experiment_design")
class ExperimentDesign {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "design_name")
    private String designName;
    @Column(name = "creator_id")
    private Long creatorId;
    @Column(name = "created_date")
    private LocalDateTime createdDate;
    @Enumerated(EnumType.STRING)
    private Status status;
    @Column(name = "experiment_steps", length = 1000)
    private String experimentSteps;
    private String reagents;
    @Column(name = "safety_notes")
    private String safetyNotes;

    public enum Status { 草稿, 已提交, 已审核 }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getDesignName() { return designName; }
    public void setDesignName(String designName) { this.designName = designName; }
    public Long getCreatorId() { return creatorId; }
    public void setCreatorId(Long creatorId) { this.creatorId = creatorId; }
    public LocalDateTime getCreatedDate() { return createdDate; }
    public void setCreatedDate(LocalDateTime createdDate) { this.createdDate = createdDate; }
    public Status getStatus() { return status; }
    public void setStatus(Status status) { this.status = status; }
    public String getExperimentSteps() { return experimentSteps; }
    public void setExperimentSteps(String experimentSteps) { this.experimentSteps = experimentSteps; }
    public String getReagents() { return reagents; }
    public void setReagents(String reagents) { this.reagents = reagents; }
    public String getSafetyNotes() { return safetyNotes; }
    public void setSafetyNotes(String safetyNotes) { this.safetyNotes = safetyNotes; }
}

// 实验执行记录实体类
@Entity
@Table(name = "experiment_record")
class ExperimentRecord {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "design_id")
    private Long designId;
    @Column(name = "executor_id")
    private Long executorId;
    @Column(name = "exec_time")
    private LocalDateTime execTime;
    private String result;
    @Column(name = "used_reagents")
    private String usedReagents;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public Long getDesignId() { return designId; }
    public void setDesignId(Long designId) { this.designId = designId; }
    public Long getExecutorId() { return executorId; }
    public void setExecutorId(Long executorId) { this.executorId = executorId; }
    public LocalDateTime getExecTime() { return execTime; }
    public void setExecTime(LocalDateTime execTime) { this.execTime = execTime; }
    public String getResult() { return result; }
    public void setResult(String result) { this.result = result; }
    public String getUsedReagents() { return usedReagents; }
    public void setUsedReagents(String usedReagents) { this.usedReagents = usedReagents; }
}

// 数据访问接口
import org.springframework.data.jpa.repository.JpaRepository;
interface ExperimentDesignRepository extends JpaRepository<ExperimentDesign, Long> {}
interface ExperimentRecordRepository extends JpaRepository<ExperimentRecord, Long> {
    List<ExperimentRecord> findByDesignId(Long designId);
}

// 服务类
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.time.LocalDateTime;
import java.util.List;

@Service
@Transactional
class ExperimentService {
    private final ExperimentDesignRepository designRepo;
    private final ExperimentRecordRepository recordRepo;

    public ExperimentService(ExperimentDesignRepository designRepo, ExperimentRecordRepository recordRepo) {
        this.designRepo = designRepo;
        this.recordRepo = recordRepo;
    }

    public ExperimentDesign createDesign(ExperimentDesign design) {
        design.setCreatedDate(LocalDateTime.now());
        design.setStatus(ExperimentDesign.Status.草稿);
        return designRepo.save(design);
    }

    public ExperimentRecord executeExperiment(ExperimentRecord record) {
        record.setExecTime(LocalDateTime.now());
        return recordRepo.save(record);
    }

    public List<ExperimentRecord> getRecordsByDesignId(Long designId) {
        return recordRepo.findByDesignId(designId);
    }
}

// 控制器
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/experiments")
class ExperimentController {
    private final ExperimentService service;

    public ExperimentController(ExperimentService service) {
        this.service = service;
    }

    @PostMapping("/designs")
    public ExperimentDesign createDesign(@RequestBody ExperimentDesign design) {
        return service.createDesign(design);
    }

    @PostMapping("/records")
    public ExperimentRecord execute(@RequestBody ExperimentRecord record) {
        return service.executeExperiment(record);
    }
}