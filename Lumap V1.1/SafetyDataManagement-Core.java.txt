// 安全数据表实体类
import javax.persistence.*;
import java.util.Date;

@Entity
@Table(name = "safety_datasheet")
class SafetyDatasheet {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "chem_id")
    private Long chemId;
    private String version;
    @Column(name = "effective_date")
    @Temporal(TemporalType.DATE)
    private Date effectiveDate;
    @Column(name = "document_path")
    private String documentPath;
    @Column(name = "file_format")
    private String fileFormat;
    @Enumerated(EnumType.STRING)
    private Status status;
    @Column(name = "created_by")
    private String createdBy;
    @Column(name = "created_time")
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdTime;

    public enum Status { 生效, 失效, 草稿 }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public Long getChemId() { return chemId; }
    public void setChemId(Long chemId) { this.chemId = chemId; }
    public String getVersion() { return version; }
    public void setVersion(String version) { this.version = version; }
    public Date getEffectiveDate() { return effectiveDate; }
    public void setEffectiveDate(Date effectiveDate) { this.effectiveDate = effectiveDate; }
    public String getDocumentPath() { return documentPath; }
    public void setDocumentPath(String documentPath) { this.documentPath = documentPath; }
    public String getFileFormat() { return fileFormat; }
    public void setFileFormat(String fileFormat) { this.fileFormat = fileFormat; }
    public Status getStatus() { return status; }
    public void setStatus(Status status) { this.status = status; }
    public String getCreatedBy() { return createdBy; }
    public void setCreatedBy(String createdBy) { this.createdBy = createdBy; }
    public Date getCreatedTime() { return createdTime; }
    public void setCreatedTime(Date createdTime) { this.createdTime = createdTime; }
}

// 数据访问接口
import org.springframework.data.jpa.repository.JpaRepository;
interface SafetyDatasheetRepository extends JpaRepository<SafetyDatasheet, Long> {
    List<SafetyDatasheet> findByChemId(Long chemId);
}

// 服务类
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.Date;
import java.util.List;

@Service
@Transactional
class SafetyDataService {
    private final SafetyDatasheetRepository repo;

    public SafetyDataService(SafetyDatasheetRepository repo) {
        this.repo = repo;
    }

    public SafetyDatasheet uploadDatasheet(SafetyDatasheet datasheet, String creator) {
        datasheet.setCreatedBy(creator);
        datasheet.setCreatedTime(new Date());
        datasheet.setStatus(SafetyDatasheet.Status.草稿);
        return repo.save(datasheet);
    }

    public List<SafetyDatasheet> getByChemId(Long chemId) {
        return repo.findByChemId(chemId);
    }

    public SafetyDatasheet updateStatus(Long id, SafetyDatasheet.Status status) {
        return repo.findById(id).map(ds -> {
            ds.setStatus(status);
            return repo.save(ds);
        }).orElseThrow(() -> new RuntimeException("Datasheet not found"));
    }
}

// 控制器
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/safety-datasheets")
class SafetyDataController {
    private final SafetyDataService service;

    public SafetyDataController(SafetyDataService service) {
        this.service = service;
    }

    @PostMapping
    public SafetyDatasheet upload(@RequestBody SafetyDatasheet datasheet, @RequestParam String creator) {
        return service.uploadDatasheet(datasheet, creator);
    }

    @GetMapping("/chem/{chemId}")
    public List<SafetyDatasheet> getByChemId(@PathVariable Long chemId) {
        return service.getByChemId(chemId);
    }
}