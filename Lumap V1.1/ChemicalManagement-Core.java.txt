// 化学品目录实体类
import javax.persistence.*;
import java.time.LocalDateTime;
import java.util.List;

@Entity
@Table(name = "chem_catalog")
class ChemCatalog {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "chemical_name", nullable = false)
    private String chemicalName;
    @Column(name = "cas_number", nullable = false, unique = true)
    private String casNumber;
    @Column(name = "molecular_formula")
    private String molecularFormula;
    @Column(name = "create_time")
    private LocalDateTime createTime;
    @Column(name = "update_time")
    private LocalDateTime updateTime;
    @Enumerated(EnumType.STRING)
    private RecordStatus recordStatus;
    @Column(name = "category_code")
    private String categoryCode;
    @Enumerated(EnumType.STRING)
    private HazardLevel hazardLevel;
    @Column(name = "storage_condition")
    private String storageCondition;

    public enum RecordStatus { 启用, 停用, 草稿 }
    public enum HazardLevel { 无危险, 低危, 中危, 高危 }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getChemicalName() { return chemicalName; }
    public void setChemicalName(String chemicalName) { this.chemicalName = chemicalName; }
    public String getCasNumber() { return casNumber; }
    public void setCasNumber(String casNumber) { this.casNumber = casNumber; }
    public String getMolecularFormula() { return molecularFormula; }
    public void setMolecularFormula(String molecularFormula) { this.molecularFormula = molecularFormula; }
    public LocalDateTime getCreateTime() { return createTime; }
    public void setCreateTime(LocalDateTime createTime) { this.createTime = createTime; }
    public LocalDateTime getUpdateTime() { return updateTime; }
    public void setUpdateTime(LocalDateTime updateTime) { this.updateTime = updateTime; }
    public RecordStatus getRecordStatus() { return recordStatus; }
    public void setRecordStatus(RecordStatus recordStatus) { this.recordStatus = recordStatus; }
    public String getCategoryCode() { return categoryCode; }
    public void setCategoryCode(String categoryCode) { this.categoryCode = categoryCode; }
    public HazardLevel getHazardLevel() { return hazardLevel; }
    public void setHazardLevel(HazardLevel hazardLevel) { this.hazardLevel = hazardLevel; }
    public String getStorageCondition() { return storageCondition; }
    public void setStorageCondition(String storageCondition) { this.storageCondition = storageCondition; }
}

// 化学品库存实体类
@Entity
@Table(name = "chem_inventory")
class ChemInventory {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(name = "chem_catalog_id")
    private Long catalogId;
    @Column(name = "inventory_quantity")
    private Double quantity;
    @Column(name = "storage_location")
    private String location;
    @Column(name = "storage_temp")
    private String temperature;
    @Column(name = "container_type")
    private String containerType;
    @Enumerated(EnumType.STRING)
    private InventoryStatus status;
    @Column(name = "last_update_time")
    private LocalDateTime updateTime;

    public enum InventoryStatus { NORMAL, BELOW_SAFE, EXPIRED }

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public Long getCatalogId() { return catalogId; }
    public void setCatalogId(Long catalogId) { this.catalogId = catalogId; }
    public Double getQuantity() { return quantity; }
    public void setQuantity(Double quantity) { this.quantity = quantity; }
    public String getLocation() { return location; }
    public void setLocation(String location) { this.location = location; }
    public String getTemperature() { return temperature; }
    public void setTemperature(String temperature) { this.temperature = temperature; }
    public String getContainerType() { return containerType; }
    public void setContainerType(String containerType) { this.containerType = containerType; }
    public InventoryStatus getStatus() { return status; }
    public void setStatus(InventoryStatus status) { this.status = status; }
    public LocalDateTime getUpdateTime() { return updateTime; }
    public void setUpdateTime(LocalDateTime updateTime) { this.updateTime = updateTime; }
}

// 数据访问接口
import org.springframework.data.jpa.repository.JpaRepository;
interface ChemCatalogRepository extends JpaRepository<ChemCatalog, Long> {
    ChemCatalog findByCasNumber(String casNumber);
}

interface InventoryRepository extends JpaRepository<ChemInventory, Long> {
    List<ChemInventory> findByCatalogId(Long catalogId);
}

// 服务类
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.time.LocalDateTime;
import java.util.List;

@Service
@Transactional
class ChemicalManagementService {
    private final ChemCatalogRepository catalogRepo;
    private final InventoryRepository inventoryRepo;

    public ChemicalManagementService(ChemCatalogRepository catalogRepo, InventoryRepository inventoryRepo) {
        this.catalogRepo = catalogRepo;
        this.inventoryRepo = inventoryRepo;
    }

    // 化学品目录操作
    public ChemCatalog createCatalog(ChemCatalog catalog) {
        catalog.setCreateTime(LocalDateTime.now());
        catalog.setRecordStatus(ChemCatalog.RecordStatus.草稿);
        return catalogRepo.save(catalog);
    }

    // 【已修复】updateCatalog 方法现在更新所有可编辑字段
    public ChemCatalog updateCatalog(Long id, ChemCatalog updated) {
        return catalogRepo.findById(id).map(catalog -> {
            catalog.setChemicalName(updated.getChemicalName());
            catalog.setMolecularFormula(updated.getMolecularFormula());
            catalog.setRecordStatus(updated.getRecordStatus());
            catalog.setCategoryCode(updated.getCategoryCode());
            catalog.setHazardLevel(updated.getHazardLevel());
            catalog.setStorageCondition(updated.getStorageCondition());
            catalog.setUpdateTime(LocalDateTime.now());
            return catalogRepo.save(catalog);
        }).orElseThrow(() -> new RuntimeException("Catalog not found"));
    }

    // 库存操作
    public ChemInventory addInventory(ChemInventory inventory) {
        inventory.setUpdateTime(LocalDateTime.now());
        return inventoryRepo.save(inventory);
    }

    // 【已修复】reduceInventory 方法现在校验 amount 是否为正数
    public ChemInventory reduceInventory(Long id, Double amount) {
        if (amount <= 0) {
            throw new RuntimeException("Amount must be positive");
        }
        return inventoryRepo.findById(id).map(inv -> {
            if (inv.getQuantity() < amount) throw new RuntimeException("Insufficient inventory");
            inv.setQuantity(inv.getQuantity() - amount);
            if (inv.getQuantity() <= 0) inv.setStatus(ChemInventory.InventoryStatus.BELOW_SAFE);
            return inventoryRepo.save(inv);
        }).orElseThrow(() -> new RuntimeException("Inventory not found"));
    }
}

// 控制器
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/chemicals")
class ChemicalManagementController {
    private final ChemicalManagementService service;

    public ChemicalManagementController(ChemicalManagementService service) {
        this.service = service;
    }

    @PostMapping("/catalogs")
    public ChemCatalog createCatalog(@RequestBody ChemCatalog catalog) {
        return service.createCatalog(catalog);
    }

    @PutMapping("/inventories/{id}/reduce")
    public ChemInventory reduceInventory(@PathVariable Long id, @RequestParam Double amount) {
        return service.reduceInventory(id, amount);
    }
}